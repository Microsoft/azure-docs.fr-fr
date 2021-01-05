---
title: 'Migrer des machines virtuelles VMware vers Azure avec le chiffrement côté serveur (SSE) et les clés gérées par le client (CMK) en utilisant Azure Migrate : Migration de serveurs'
description: 'Découvrez comment migrer des machines virtuelles VMware vers Azure avec le chiffrement côté serveur (SSE) et les clés gérées par le client (CMK) en utilisant Azure Migrate : Migration de serveurs'
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 03/12/2020
ms.openlocfilehash: 8a174c3b2bfb390eb7d691ae1bdcb0e28dde9032
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751085"
---
# <a name="migrate-vmware-vms-to-azure-vms-enabled-with-server-side-encryption-and-customer-managed-keys"></a>Migrer des machines virtuelles VMware vers des machines virtuelles Azure configurées pour le chiffrement côté serveur et les clés gérées par le client

Cet article explique comment migrer des machines virtuelles VMware vers des machines virtuelles Azure avec des disques chiffrés à l’aide du chiffrement côté serveur (SSE) et avec des clés gérées par le client (CMK), en utilisant Azure Migrate : Migration de serveurs (réplication sans agent).

L’expérience du portail Azure Migrate : Migration de serveurs vous permet de [migrer des machines virtuelles VMware vers Azure avec la réplication sans agent.](tutorial-migrate-vmware.md) L’expérience du portail n’offre actuellement pas la possibilité d’activer le chiffrement côté serveur avec des clés gérées par le client pour vos disques répliqués dans Azure. La possibilité d’activer le chiffrement côté serveur avec des clés gérées par le client pour les disques répliqués est actuellement disponible seulement via l’API REST. Dans cet article, vous allez découvrir comment créer et déployer un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) pour répliquer une machine virtuelle VMware et pour configurer les disques répliqués dans Azure pour l’utilisation du chiffrement côté serveur avec des clés gérées par le client.

Les exemples de cet article utilisent [Azure PowerShell](/powershell/azure/new-azureps-module-az) pour effectuer les tâches nécessaires à la création et au déploiement du modèle Resource Manager.

[Découvrez plus d’informations](../virtual-machines/disk-encryption.md) sur le chiffrement côté serveur avec des clés gérées par le client pour les disques managés.

## <a name="prerequisites"></a>Prérequis

- [Passez en revue le tutoriel](tutorial-migrate-vmware.md) sur la migration de machines virtuelles VMware vers Azure avec la réplication sans agent pour comprendre les exigences des outils.
- [Suivez ces instructions](./create-manage-projects.md) pour créer un projet Azure Migrate et ajouter l’outil **Azure Migrate : Migration de serveurs** au projet.
- [Suivez ces instructions](how-to-set-up-appliance-vmware.md) pour configurer l’appliance Azure Migrate pour VMware dans votre environnement local et pour effectuer la découverte.

## <a name="prepare-for-replication"></a>Préparer pour la réplication

Une fois la découverte des machines virtuelles terminée, la ligne Serveurs découverts sur la vignette Migration de serveurs affiche le nombre de machines virtuelles VMware découvertes par l’appliance.

Avant de pouvoir commencer à répliquer des machines virtuelles, l’infrastructure de réplication doit être préparée.

1. Créez une instance Service Bus dans la région cible. L’instance Service Bus est utilisée par l’appliance Azure Migrate locale pour communiquer avec le service Migration de serveurs pour coordonner la réplication et la migration.
2. Créez un compte de stockage pour le transfert des journaux des opérations de la réplication.
3. Créez un compte de stockage dans lequel l’appliance Azure Migrate charge les données de réplication.
4. Créez un coffre de clés et configurez-le pour gérer les jetons de signature d’accès partagé pour l’accès aux objets blob sur les comptes de stockage créés à l’étape 3 et 4.
5. Générez un jeton de signature d’accès partagé pour l’instance Service bus créée à l’étape 1 et créez un secret pour le jeton dans le coffre de clés créé à l’étape précédente.
6. Créez une stratégie d’accès Key Vault pour permettre à l’appliance Azure Migrate locale (avec l’application AAD de l’appliance) et au service Migration de serveurs d’accéder au coffre de clés.
7. Créez une stratégie de réplication et configurez le service Migration de serveurs avec les informations détaillées de l’infrastructure de réplication créée à l’étape précédente.

L’infrastructure de réplication doit être créée dans la région Azure cible pour la migration et dans l’abonnement Azure cible vers lesquels les machines virtuelles doivent être migrées.

L’expérience du portail Migration de serveurs simplifie la préparation de l’infrastructure de réplication en faisant automatiquement ceci pour vous quand vous répliquez une machine virtuelle pour la première fois dans un projet. Dans cet article, nous partons du principe que vous avez déjà répliqué une ou plusieurs machines virtuelles avec l’expérience du portail et que l’infrastructure de réplication est déjà créée. Nous allons voir comment découvrir les informations détaillées de l’infrastructure de réplication existante et comment les utiliser comme entrées du modèle Resource Manager qui sera utilisé pour configurer la réplication avec les clés gérées par le client.

### <a name="identifying-replication-infrastructure-components"></a>Identification des composants de l’infrastructure de réplication

1. Dans le portail Azure, accédez à la page des groupes de ressources et sélectionnez le groupe de ressources où le projet Azure Migrate a été créé.
2. Sélectionnez **Déploiements** dans le menu de gauche et recherchez un nom de déploiement commençant par la chaîne *« Microsoft.MigrateV2.VMwareV2EnableMigrate »* . Vous verrez une liste de modèles Resource Manager créés par l’expérience du portail pour configurer la réplication pour les machines virtuelles de ce projet. Nous allons télécharger un de ces modèles et l’utiliser comme base afin de préparer le modèle pour la réplication avec des clés gérées par le client.
3. Pour télécharger le modèle, sélectionnez un déploiement correspondant au modèle de chaîne de l’étape précédente > Sélectionnez **Modèle** dans le menu de gauche > Cliquez sur **Télécharger** dans le menu supérieur. Enregistrez le fichier template.json localement. Vous allez modifier ce fichier de modèle à la dernière étape.

## <a name="create-a-disk-encryption-set"></a>Créer un jeu de chiffrement de disque

Un objet de jeu de chiffrement de disque mappe les disques managés à un coffre de clés contenant les clés gérées par le client à utiliser pour le chiffrement côté serveur. Pour répliquer des machines virtuelles avec des clés gérées par le client, vous allez créer un jeu de chiffrement de disque et le passer comme entrée à l’opération de réplication.

Suivez l’exemple [ici](../virtual-machines/windows/disks-enable-customer-managed-keys-powershell.md) pour créer un jeu de chiffrement de disque avec Azure PowerShell. Vérifiez que le jeu de chiffrement de disque est créé dans l’abonnement cible vers lequel les machines virtuelles doivent être migrées et dans la région Azure cible de la migration.

Le jeu de chiffrement de disque peut être configuré pour chiffrer les disques managés avec une clé gérée par le client, ou pour le double chiffrement avec une clé gérée par le client et une clé de plateforme. Pour utiliser l’option double chiffrement au repos, configurez le chiffrement de disque défini comme décrit [ici](../virtual-machines/windows/disks-enable-double-encryption-at-rest-powershell.md).

Dans l’exemple ci-dessous, l’ensemble de chiffrement du disque est configuré pour utiliser une clé gérée par le client.

```azurepowershell
$Location = "southcentralus"                           #Target Azure region for migration 
$TargetResourceGroupName = "ContosoMigrationTarget"
$KeyVaultName = "ContosoCMKKV"
$KeyName = "ContosoCMKKey"
$KeyDestination = "Software"
$DiskEncryptionSetName = "ContosoCMKDES"

$KeyVault = New-AzKeyVault -Name $KeyVaultName -ResourceGroupName $TargetResourceGroupName -Location $Location -EnableSoftDelete -EnablePurgeProtection

$Key = Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $KeyName -Destination $KeyDestination

$desConfig = New-AzDiskEncryptionSetConfig -Location $Location -SourceVaultId $KeyVault.ResourceId -KeyUrl $Key.Key.Kid -IdentityType SystemAssigned

$des = New-AzDiskEncryptionSet -Name $DiskEncryptionSetName -ResourceGroupName $TargetResourceGroupName -InputObject $desConfig

Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get

New-AzRoleAssignment -ResourceName $KeyVaultName -ResourceGroupName $TargetResourceGroupName -ResourceType "Microsoft.KeyVault/vaults" -ObjectId $des.Identity.PrincipalId -RoleDefinitionName "Reader"
```

## <a name="get-details-of-the-vmware-vm-to-migrate"></a>Obtenir les informations détaillées de la machine virtuelle VMware à migrer

Dans cette étape, vous allez utiliser Azure PowerShell pour obtenir les informations détaillées de la machine virtuelle qui doit être migrée. Ces informations seront utilisées pour construire le modèle Resource Manager pour la réplication. Plus précisément, les deux propriétés intéressantes sont :

- L’ID de ressource des machines pour les machines virtuelles découvertes.
- La liste des disques de la machine virtuelle et de leurs identificateurs de disque.

```azurepowershell

$ProjectResourceGroup = "ContosoVMwareCMK"   #Resource group that the Azure Migrate Project is created in
$ProjectName = "ContosoVMwareCMK"            #Name of the Azure Migrate Project

$solution = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.Migrate/MigrateProjects/solutions -ExpandProperties -ResourceName $ProjectName | where Name -eq "Servers-Discovery-ServerDis
covery"

# Displays one entry for each appliance in the project mapping the appliance to the VMware sites discovered through the appliance.
$solution.Properties.details.extendedDetails.applianceNameToSiteIdMapV2 | ConvertFrom-Json | select ApplianceName, SiteId
```
```Output
ApplianceName  SiteId
-------------  ------
VMwareApplianc /subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite
```

Copiez la valeur de la chaîne SiteId correspondant à l’appliance Azure Migrate via laquelle la machine virtuelle est découverte. Dans l’exemple ci-dessus, le SiteId est *« /subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite »*

```azurepowershell

#Replace value with SiteId from the previous step
$SiteId = "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite"
$SiteName = Get-AzResource -ResourceId $SiteId -ExpandProperties | Select-Object -ExpandProperty Name
$DiscoveredMachines = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.OffAzure/VMwareSites/machines  -ExpandProperties -ResourceName $SiteName

#Get machine details
PS /home/bharathram> $MachineName = "FPL-W19-09"     #Replace string with VMware VM name of the machine to migrate
PS /home/bharathram> $machine = $Discoveredmachines | where {$_.Properties.displayName -eq $MachineName}
PS /home/bharathram> $machine.count   #Validate that only 1 VM was found matching this name.
```

Copiez les valeurs ResourceId, Name et UUID de disque de la machine à migrer.
```Output
PS > $machine.Name
10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210
PS > $machine.ResourceId
/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210

PS > $machine.Properties.disks | select uuid, label, name, maxSizeInBytes

uuid                                 label       name    maxSizeInBytes
----                                 -----       ----    --------------
6000C291-5106-2aac-7a74-4f33c3ddb78c Hard disk 1 scsi0:0    42949672960
6000C293-39a1-bd70-7b24-735f0eeb79c4 Hard disk 2 scsi0:1    53687091200
6000C29e-cbee-4d79-39c7-d00dd0208aa9 Hard disk 3 scsi0:2    53687091200

```

## <a name="create-resource-manager-template-for-replication"></a>Créer un modèle Resource Manager pour la réplication

- Ouvrez le fichier de modèle Resource Manager que vous avez téléchargé à l’étape **Identifier les composants de l’infrastructure de réplication** dans l’éditeur de votre choix.
- Supprimez toutes les définitions de ressource du modèle, à l’exception des ressources de type *« Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems »* .
- S’il existe plusieurs définitions de ressource de ce type, supprimez-les toutes sauf une. Supprimez les définitions de propriété **dependsOn** de la définition de ressource.
- À la fin de cette étape, vous devez avoir un fichier ressemblant à l’exemple ci-dessous et ayant le même ensemble de propriétés.

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoMigration7371rsvault/VMware104e4replicationfabric/VMware104e4replicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
            "properties": {
                "policyId": "/Subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.RecoveryServices/vaults/ContosoMigration7371rsvault/replicationPolicies/migrateVMware104e4sitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
                    "targetResourceGroupId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG",
                    "targetNetworkId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG/providers/Microsoft.Network/virtualNetworks/PayrollNW",
                    "targetSubnetName": "PayrollSubnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C295-dafe-a0eb-906e-d47cb5b05a1d",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                            "logStorageAccountSasSecretName": "migratelsa1432469187-cacheSas",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                    "targetVmName": "PayrollWeb04"
                }
            }
        }
    ]
}
```

- Éditez la propriété **name** dans la définition de ressource. Remplacez la chaîne qui suit le dernier caractère « / » dans la propriété name par la valeur de *$machine.Name* (de l’étape précédente).
- Remplacez la valeur de la propriété **properties.providerSpecificDetails.vmwareMachineId** par la valeur de *$machine.ResourceId* (de l’étape précédente).
- Définissez les valeurs de **targetResourceGroupId**, **targetNetworkId** et **targetSubnetName** respectivement sur l’ID du groupe de ressources cible, sur l’ID de ressource du réseau virtuel cible et sur le nom de sous-réseau cible.
- Définissez la valeur de **licenseType** sur « WindowsServer » afin d’appliquer Azure Hybrid Benefit pour cette machine virtuelle. Si cette machine virtuelle n’est pas éligible à Azure Hybrid Benefit, définissez la valeur de **licenseType** sur NoLicenseType.
- Remplacez la valeur de la propriété **targetVmName** par le nom de la machine virtuelle Azure souhaitée pour la machine virtuelle migrée.
- Vous pouvez ajouter une propriété nommée **targetVmSize** sous la propriété **targetVmName**. Définissez la valeur de la propriété **targetVmSize** sur la taille de la machine virtuelle Azure souhaitée pour la machine virtuelle migrée.
- La propriété **disksToInclude** est une liste d’entrées de disque pour la réplication, chaque élément de la liste représentant un disque local. Créez autant d’éléments de liste que le nombre de disques sur la machine virtuelle locale. Remplacez la propriété **diskId** de l’élément de liste par l’UUID des disques identifiés à l’étape précédente. Définissez **isOSDisk** sur la valeur « true » pour le disque du système d’exploitation de la machine virtuelle et sur « false » pour tous les autres disques. Laissez les propriétés **logStorageAccountId** et **logStorageAccountSasSecretName** inchangées. Définissez la valeur de **diskType** sur le type Disque managé Azure (*Standard_LRS, Premium_LRS, StandardSSD_LRS*) à utiliser pour le disque. Pour les disques qui doivent être chiffrés avec des clés gérées par le client, ajoutez une propriété nommée **diskEncryptionSetId** et définissez la valeur sur l’ID de ressource du jeu de chiffrement de disque ( **$des.Id**) à l’étape *Créer un jeu de chiffrement de disque*.
- Enregistrez le fichier de modèle modifié. Dans l’exemple ci-dessus, le fichier de modèle modifié se présente comme suit :

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoVMwareCMK00ddrsvault/VMwareApplianca8bareplicationfabric/VMwareApplianca8bareplicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
            "properties": {
                "policyId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.RecoveryServices/vaults/ContosoVMwareCMK00ddrsvault/replicationPolicies/migrateVMwareApplianca8basitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
                    "targetResourceGroupId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoMigrationTarget",
                    "targetNetworkId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/cmkRTest/providers/Microsoft.Network/virtualNetworks/cmkvm1_vnet",
                    "targetSubnetName": "cmkvm1_subnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C291-5106-2aac-7a74-4f33c3ddb78c",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C293-39a1-bd70-7b24-735f0eeb79c4",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C29e-cbee-4d79-39c7-d00dd0208aa9",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                    "performAutoResync": "true",
                    "targetVmName": "FPL-W19-09"
                }
            }
        }
    ]
}
```

## <a name="set-up-replication"></a>Configurer la réplication

Vous pouvez maintenant déployer le modèle Resource Manager modifié dans le groupe de ressources du projet afin de configurer la réplication pour la machine virtuelle. Découvrez comment [déployer une ressource avec des modèles Resource Manager et Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName $ProjectResourceGroup -TemplateFile "C:\Users\Administrator\Downloads\template.json"
```

```Output
DeploymentName          : template
ResourceGroupName       : ContosoVMwareCMK
ProvisioningState       : Succeeded
Timestamp               : 3/11/2020 8:52:00 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
Outputs                 :
DeploymentDebugLogLevel :

```

## <a name="next-steps"></a>Étapes suivantes

[Superviser l’état de la réplication](tutorial-migrate-vmware.md#track-and-monitor) via l’expérience du portail, et effectuer des test de migration et la migration.