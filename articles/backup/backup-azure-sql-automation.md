---
title: Sauvegarde et restauration d’une base de données SQL dans une machine virtuelle Azure via PowerShell
description: Sauvegardez et restaurez des bases de données SQL dans des machines virtuelles Azure à l’aide de Sauvegarde Azure et PowerShell.
ms.topic: conceptual
ms.date: 03/15/2019
ms.assetid: 57854626-91f9-4677-b6a2-5d12b6a866e1
ms.openlocfilehash: 0a3467ffa3a67ac9ad593748948cea8da59e3e6b
ms.sourcegitcommit: f7084d3d80c4bc8e69b9eb05dfd30e8e195994d8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97734536"
---
# <a name="back-up-and-restore-sql-databases-in-azure-vms-with-powershell"></a>Sauvegarder et restaurer des bases de données SQL dans des machines virtuelles Azure à l’aide de PowerShell

Cet article explique comment utiliser Azure PowerShell pour sauvegarder et restaurer une base de données SQL au sein d’une machine virtuelle Azure à l’aide du coffre Recovery Services [Sauvegarde Azure](backup-overview.md).

Cet article explique comment :

> [!div class="checklist"]
>
> * Configurer PowerShell et inscrire le fournisseur Azure Recovery Services.
> * Créez un coffre Recovery Services.
> * Configurer la sauvegarde pour la base de données SQL au sein d’une machine virtuelle Azure.
> * Exécuter un travail de sauvegarde.
> * Restaurer une base de données SQL sauvegardée.
> * Suivre des travaux de sauvegarde et de restauration.

## <a name="before-you-start"></a>Avant de commencer

* [En savoir plus](backup-azure-recovery-services-vault-overview.md) sur les coffres Recovery Services.
* Découvrez les fonctionnalités pour [sauvegarder les bases de données SQL dans des machines virtuelles Azure](backup-azure-sql-database.md#before-you-start).
* Passez en revue la hiérarchie des objets PowerShell pour Recovery Services.

### <a name="recovery-services-object-hierarchy"></a>Hiérarchie des objets dans Recovery Services

La hiérarchie des objets est résumée dans le schéma suivant.

![Hiérarchie des objets dans Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Passez en revue la [référence sur la cmdlet](/powershell/module/az.recoveryservices) **Az.RecoveryServices** dans la bibliothèque Azure.

### <a name="set-up-and-install"></a>Configurer et installer

Configurez PowerShell comme suit :

1. [Téléchargez la dernière version d’Az PowerShell](/powershell/azure/install-az-ps). La version 1.5.0 est la version minimale requise.

2. Recherchez les cmdlets PowerShell Sauvegarde Azure avec cette commande :

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Passez en revue les alias et cmdlets pour Sauvegarde Azure et le coffre Recovery Services. Voici un exemple de résultat possible. Il ne s’agit pas d’une liste complète des cmdlets.

    ![Liste des cmdlets Recovery Services](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Connectez-vous à votre compte Azure à l’aide de **Connect-AzAccount**.
5. Sur la page web qui s’affiche, vous êtes invité à entrer les informations d’identification de votre compte.

    * Vous pouvez également inclure les informations d’identification de votre compte en tant que paramètres dans la cmdlet **Connect-AzAccount** avec **-Credential**.
    * Si vous êtes un partenaire CSP travaillant pour un locataire, spécifiez le client en tant que locataire à l’aide de son ID locataire ou de son nom de domaine principal. Par exemple **Connect-AzAccount -Tenant** fabrikam.com.

6. Associez l’abonnement que vous souhaitez utiliser avec le compte, car un compte peut compter plusieurs abonnements.

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Si vous utilisez le service Sauvegarde Azure pour la première fois, utilisez la cmdlet **Register-AzResourceProvider** pour associer le fournisseur Azure Recovery Service à votre abonnement.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Vérifiez que les fournisseurs ont été correctement inscrits :

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. Dans la sortie de commande, vérifiez que **RegistrationState** a maintenant la valeur **Inscrit**. Si ce n’est pas ce cas, réexécutez la cmdlet **Register-AzResourceProvider**.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Pour créer un coffre Recovery Services, procédez comme suit :

Le coffre Recovery Services étant une ressource Resource Manager, vous devez le placer dans un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un avec la cmdlet **New-AzResourceGroup**. Quand vous créez un groupe de ressources, spécifiez son nom et son emplacement.

1. Un coffre est placé dans un groupe de ressources. Si vous n’avez pas de groupe de ressources, créez-en un avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Dans l’exemple suivant, nous créons un groupe de ressources dans la région USA Ouest.

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Utilisez la cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) pour créer le coffre. Spécifiez pour le coffre le même emplacement que pour le groupe de ressources.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Spécifiez le type de redondance à utiliser pour le stockage de coffre.

    * Vous pouvez utiliser le [stockage localement redondant](../storage/common/storage-redundancy.md#locally-redundant-storage), le [stockage géoredondant](../storage/common/storage-redundancy.md#geo-redundant-storage) ou le [stockage redondant interzone](../storage/common/storage-redundancy.md#zone-redundant-storage).
    * L’exemple suivant définit l’option **-BackupStorageRedundancy** pour la cmdlet [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) de **testvault** définie sur **GeoRedundant**.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Afficher les coffres dans un abonnement

Pour afficher tous les coffres de l’abonnement, utilisez [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault).

```powershell
Get-AzRecoveryServicesVault
```

Le résultat ressemble à ce qui suit. Le groupe de ressources et l’emplacement associés sont fournis.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>Définir le contexte du coffre

Stockez l’objet de coffre dans une variable et définissez le contexte du coffre.

* Étant donné que de nombreuses cmdlets Sauvegarde Azure exigent l’objet de coffre Recovery Services en tant qu’entrée, il est judicieux de le stocker dans une variable.
* Le contexte du coffre spécifie le type de données protégées dans le coffre. Définissez-le avec [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). Une fois le contexte défini, il s’applique à toutes les cmdlets suivantes.

L’exemple suivant définit le contexte pour le coffre **testvault**.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Récupérer l’ID de coffre

Conformément aux instructions d’Azure PowerShell, nous prévoyons de déprécier la définition du contexte de coffre. À la place, vous pouvez stocker ou récupérer l’ID du coffre et le transmettre aux commandes appropriées, comme suit :

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

## <a name="configure-a-backup-policy"></a>Configurer une stratégie de sauvegarde

Une stratégie de sauvegarde spécifie la planification des sauvegardes et la durée de conservation des points de récupération de sauvegarde :

* Une stratégie de sauvegarde est associée à au moins une stratégie de rétention. Une stratégie de rétention définit la durée de conservation d’un point de restauration avant sa suppression.
* Affichez la stratégie de rétention de sauvegarde par défaut avec [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Affichez la planification de la stratégie de sauvegarde par défaut avec [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* Vous utilisez la cmdlet [New-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) pour créer une stratégie de sauvegarde. Vous entrez les objets de stratégie de planification et de rétention.

Par défaut, une heure de début est définie dans l’objet de la stratégie de planification. Utilisez l’exemple suivant pour modifier l’heure de début sur l’heure de début souhaitée. L’heure de début souhaitée doit également être au format UTC. L’exemple suivant suppose que l’heure de début souhaitée est 01:00 UTC pour les sauvegardes quotidiennes.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Vous devez fournir l’heure de début en multiples de 30 minutes uniquement. Dans l’exemple ci-dessus, il peut s’agir uniquement de « 01:00:00 » ou de « 02:30:00 ». L’heure de début ne peut pas être « 01:15:00 ».

L’exemple suivant stocke la stratégie de planification et la stratégie de conservation dans des variables. Il utilise ensuite ces variables en tant que paramètres pour une nouvelle stratégie (**NewSQLPolicy**). **NewSQLPolicy** effectue une sauvegarde quotidienne « complète », la conserve pendant 180 jours et effectue une sauvegarde de journal toutes les 2 heures

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "MSSQL"
$NewSQLPolicy = New-AzRecoveryServicesBackupProtectionPolicy -Name "NewSQLPolicy" -WorkloadType "MSSQL" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

Le résultat ressemble à ce qui suit.

```output
Name                 WorkloadType       BackupManagementType BackupTime                Frequency                                IsDifferentialBackup IsLogBackupEnabled
                                                                                                                                Enabled
----                 ------------       -------------------- ----------                ---------                                -------------------- ------------------
NewSQLPolicy         MSSQL              AzureWorkload        3/15/2019 01:30:00 AM      Daily                                    False                True
```

## <a name="enable-backup"></a>Activer la sauvegarde

### <a name="registering-the-sql-vm"></a>Inscription de la machine virtuelle SQL

Pour les sauvegardes de machines virtuelles Azure et les partages de fichiers Azure, le service Sauvegarde peut se connecter à ces ressources Azure Resource Manager et récupérer les informations pertinentes. Étant donné que SQL est une application au sein d’une machine virtuelle Azure, le service Sauvegarde a besoin d’autorisations pour accéder à l’application et récupérer les informations nécessaires. Pour ce faire, vous devez *« inscrire »* la machine virtuelle Azure qui contient l’application SQL auprès d’un coffre Recovery Services. Quand vous avez inscrit une machine virtuelle SQL avec un coffre, vous pouvez protéger les bases de données de ce coffre uniquement. Utilisez l’applet de commande PowerShell [Register-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer) pour inscrire la machine virtuelle.

```powershell
 $myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

La commande renvoie un « conteneur de sauvegarde »de cette ressource et l’état est défini sur « inscrit »

> [!NOTE]
> Si le paramètre Force n’est pas spécifié, vous êtes invité à confirmer avec un texte « Do you want to disable protection for this container » (Voulez-vous désactiver la protection de ce conteneur). Veuillez ignorer ce texte et indiquez « O » pour confirmer. Il s’agit d’un problème connu et nous travaillons à sa résolution en supprimant le texte et la nécessité d’avoir le paramètre Force.

### <a name="fetching-sql-dbs"></a>Récupération des bases de données SQL

Une fois l’inscription effectuée, le service Sauvegarde est en mesure de répertorier tous les composants SQL disponibles au sein de la machine virtuelle. Pour afficher tous les composants SQL restant à sauvegarder pour ce coffre, utilisez l’applet de commande PowerShell [Get-AzRecoveryServicesBackupProtectableItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem)

```powershell
Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -VaultId $targetVault.ID
```

La sortie affiche tous les composants SQL non protégés sur toutes les machines virtuelles SQL inscrites dans ce coffre avec les éléments ItemType et ServerName. Vous pouvez filtrer davantage sur une machine virtuelle en particulier en transmettant le paramètre « -Container » ou utilisez la combinaison de « Name » et « ServerName » avec l’indicateur ItemType pour arriver à un élément SQL unique.

```powershell
$SQLDB = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID -Name "<Item Name>" -ServerName "<Server Name>"
```

### <a name="configuring-backup"></a>Configuration de la sauvegarde

Maintenant que nous disposons de la base de données SQL requise et de la stratégie avec laquelle elle doit être sauvegardée, nous pouvons utiliser la cmdlet [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) pour configurer la sauvegarde pour cette base de données SQL.

```output
Enable-AzRecoveryServicesBackupProtection -ProtectableItem $SQLDB -Policy $NewSQLPolicy
```

La commande attend la fin de la configuration de la sauvegarde et renvoie le résultat suivant.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 6:00:21 PM      3/18/2019 6:01:35 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="fetching-new-sql-dbs"></a>Récupération de nouvelles bases de données SQL

Une fois la machine inscrite, le service Sauvegarde récupère les informations des bases de données disponibles. Si des bases de données SQL ou des instances SQL sont ajoutées ultérieurement à la machine inscrite, vous devez déclencher manuellement le service de sauvegarde pour effectuer une nouvelle « recherche » et obtenir de nouveau **toutes** les bases de données non protégées (y compris celles qui viennent d’être ajoutées). Utilisez l’applet de commande PowerShell [Initialize-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/initialize-azrecoveryservicesbackupprotectableitem) sur la machine virtuelle SQL pour effectuer une nouvelle recherche. La commande attend la fin de l’opération. Utilisez ultérieurement l’applet de commande PowerShell [Get-AzRecoveryServicesBackupProtectableItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem) pour obtenir la liste des derniers composants SQL non protégés.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Initialize-AzRecoveryServicesBackupProtectableItem -Container $SQLContainer -WorkloadType MSSQL -VaultId $targetvault.ID
Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID
```

Une fois que les éléments protégeables pertinents sont récupérés, activez les sauvegardes comme indiqué dans la [section ci-dessus](#configuring-backup).
Si vous ne souhaitez pas détecter manuellement les nouvelles bases de données, vous pouvez choisir l’autoprotection, comme expliqué [ci-dessous](#enable-autoprotection).

## <a name="enable-autoprotection"></a>Activer l’autoprotection

Vous pouvez configurer la sauvegarde pour que toutes les bases de données ajoutées à l’avenir soient automatiquement protégées avec une stratégie spécifique. Pour activer l’autoprotection, utilisez l’applet de commande PowerShell [Enable-AzRecoveryServicesBackupAutoProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupautoprotection).

Puisque l’instruction est de sauvegarder toutes les futures bases de données, l’opération est effectuée à un niveau SQLInstance.

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Enable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -Policy $NewSQLPolicy -VaultId $targetvault.ID
```

Une fois l’intention d’autoprotection donnée, la demande faite à la machine de récupérer les bases de données nouvellement ajoutées se met en place sous la forme d’une tâche en arrière-plan programmée qui s’exécute toutes les 8 heures.

## <a name="restore-sql-dbs"></a>Restaurer les bases de données SQL

Sauvegarde Azure peut restaurer des bases de données SQL Server s’exécutant sur des machines virtuelles Azure comme suit :

* Restaurer à une date ou une heure spécifique (à la seconde), en utilisant les sauvegardes de fichiers journaux. Sauvegarde Azure détermine automatiquement la sauvegarde différentielle complète appropriée, et la chaîne des sauvegardes de fichiers journaux nécessaires pour restaurer en fonction de la date/heure sélectionnée.
* Restaurez une sauvegarde complète ou différentielle spécifique sur un point de récupération spécifique.

Vérifiez les prérequis mentionnés [ici](restore-sql-database-azure-vm.md#restore-prerequisites) avant de restaurer des bases de données SQL.

Commencez par récupérer la base de données SQL sauvegardée appropriée à l’aide de l’applet de commande PowerShell [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
```

### <a name="fetch-the-relevant-restore-time"></a>Récupérer l’heure de restauration pertinente

Comme indiqué plus haut, vous pouvez restaurer la base de données SQL sauvegardée vers une copie complète/différentielle **OU** un point dans le temps de journal.

#### <a name="fetch-distinct-recovery-points"></a>Récupérer des points de récupération distincts

Utilisez [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) pour récupérer des points de récupération (différentielle/complète) distincts pour une base de données SQL sauvegardée.

```powershell
$startDate = (Get-Date).AddDays(-7).ToUniversalTime()
$endDate = (Get-Date).ToUniversalTime()
Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -StartDate $startdate -EndDate $endDate
```

Le résultat ressemble à l’exemple suivant

```output
RecoveryPointId    RecoveryPointType  RecoveryPointTime      ItemName                             BackupManagemen
                                                                                                  tType
---------------    -----------------  -----------------      --------                             ---------------
6660368097802      Full               3/18/2019 8:09:35 PM   MSSQLSERVER;model             AzureWorkload
```

Pour récupérer le point de récupération approprié, utilisez le filtre « RecoveryPointId » ou un filtre de tableau.

```powershell
$FullRP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -RecoveryPointId "6660368097802"
```

#### <a name="fetch-point-in-time-recovery-point"></a>Récupérer un point de récupération jusqu’à une date et heure

Si vous souhaitez restaurer la base de données à un certain point dans le temps, utilisez l’applet de commande PowerShell [Get-AzRecoveryServicesBackupRecoveryLogChain](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverylogchain). La cmdlet renvoie une liste de dates qui représentent les heures de début et de fin d’une séquence de journaux de transactions consécutifs continue pour cet élément de sauvegarde SQL. Le point dans le temps souhaité doit être dans cette plage.

```powershell
Get-AzRecoveryServicesBackupRecoveryLogChain -Item $bkpItem -VaultId $targetVault.ID
```

Vous devez obtenir un résultat semblable à l’exemple qui suit.

```output
ItemName                       StartTime                      EndTime
--------                       ---------                      -------
SQLDataBase;MSSQLSERVER;azu... 3/18/2019 8:09:35 PM           3/19/2019 12:08:32 PM
```

La sortie ci-dessus signifie que vous pouvez effectuer une restauration à n’importe quel point dans le temps entre l’heure de début et l’heure de fin affichées. Les heures sont exprimées en UTC. Créez n’importe quel point dans le temps dans PowerShell qui se trouve dans la plage indiquée ci-dessus.

> [!NOTE]
> Quand un point dans le temps de journal est sélectionné pour la restauration, vous n’avez pas besoin de spécifier le point de départ, c’est-à-dire la sauvegarde complète à partir de laquelle la base de données est restaurée. Le service de sauvegarde Azure s’occupe de la totalité du plan de récupération (la sauvegarde complète à choisir, les sauvegardes de fichier journal à appliquer, etc.).

### <a name="determine-recovery-configuration"></a>Déterminer la configuration de récupération

Dans le cas d’une restauration de bases de données SQL, les scénarios de restauration suivants sont pris en charge.

* Remplacement de la base de données SQL sauvegardée par des données d’un autre point de récupération - OriginalWorkloadRestore
* Restauration de la base de données SQL en tant que nouvelle base de données dans la même instance SQL - AlternateWorkloadRestore
* Restauration de la base de données SQL en tant que nouvelle base de données dans une autre machine virtuelle SQL - AlternateWorkloadRestore
* Restauration de la base de données SQL en tant que fichiers.bak - RestoreAsFiles

Après la récupération du point de récupération approprié (distinct ou point dans le temps de journal), utilisez l’applet de commande PowerShell [Get-AzRecoveryServicesBackupWorkloadRecoveryConfig](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) pour récupérer l’objet de configuration de récupération conformément au plan de récupération souhaité.

#### <a name="original-workload-restore"></a>Restauration de la charge de travail d’origine

Pour remplacer la base de données sauvegardée par des données du point de récupération, spécifiez simplement l’indicateur approprié et le point de récupération pertinent, comme indiqué dans les exemples suivants.

##### <a name="original-restore-with-distinct-recovery-point"></a>Restauration d’origine avec un point de récupération distinct

```powershell
$OverwriteWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -OriginalWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="original-restore-with-log-point-in-time"></a>Restauration d’origine avec un point dans le temps de journal

```powershell
$OverwriteWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem  -OriginalWorkloadRestore -VaultId $targetVault.ID
```

#### <a name="alternate-workload-restore"></a>Restauration de charge de travail alternative

> [!IMPORTANT]
> Une base de données SQL sauvegardée peut être restaurée en tant que nouvelle base de données d’une autre SQLInstance uniquement, dans une machine virtuelle Azure « inscrite » dans ce coffre.

Comme indiqué ci-dessus, si la SQLInstance cible se trouve dans une autre machine virtuelle Azure, assurez-vous qu’elle est [inscrite auprès de ce coffre](#registering-the-sql-vm) et que la SQLInstance pertinente apparaît en tant qu’élément pouvant être protégé.

```powershell
$TargetInstance = Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -ItemType SQLInstance -Name "<SQLInstance Name>" -ServerName "<SQL VM name>" -VaultId $targetVault.ID
```

Puis, transmettez simplement le point de récupération pertinent, l’instance SQL cible avec l’indicateur approprié comme indiqué ci-dessous.

##### <a name="alternate-restore-with-distinct-recovery-point"></a>Restauration alternative avec un point de récupération distinct

```powershell
$AnotherInstanceWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetItem $TargetInstance -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="alternate-restore-with-log-point-in-time"></a>Restauration alternative avec un point dans le temps de journal

```powershell
$AnotherInstanceWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="restore-as-files"></a>Restaurer sous forme de fichiers

Pour restaurer les données de sauvegarde sous forme de fichiers .bak plutôt que sous forme de base de données, choisissez l’option **Restaurer sous forme de fichiers**. La base de données SQL sauvegardée peut être restaurée sur n’importe quelle machine virtuelle cible qui est inscrite auprès de ce coffre.

```powershell
$TargetContainer= Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName "VM name" -VaultId $vaultID
```

##### <a name="restore-as-files-with-distinct-recovery-point"></a>Restauration sous forme de fichiers avec un point de récupération distinct

```powershell
$FileRestoreWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-latest-full"></a>Restaurer sous forme de fichiers avec un point d’enregistrement dans le journal à partir de la dernière sauvegarde complète

```powershell
$FileRestoreWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-a-specified-full"></a>Restaurer sous forme de fichiers avec un point d’enregistrement dans le journal à partir de la sauvegarde complète spécifiée

Si vous souhaitez fournir une sauvegarde complète spécifique qui doit être utilisée pour la restauration, utilisez la commande suivante :

```powershell
$FileRestoreWithLogAndSpecificFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -FromFull $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

L’objet de configuration de point de récupération final obtenu avec l’applet de commande PowerShell [Get-AzRecoveryServicesBackupWorkloadRecoveryConfig](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) a toutes les informations pertinentes pour la restauration et est illustré ci-dessous.

```output
TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : No
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
```

Vous pouvez modifier le nom de la base de données restaurée, ainsi que les champs OverwriteWLIfpresent, NoRecoveryMode, et targetPhysicalPath. Obtenez plus de détails sur les chemins d’accès de fichier cible comme indiqué ci-dessous.

```powershell
$AnotherInstanceWithFullConfig.targetPhysicalPath
```

```output
MappingType SourceLogicalName SourcePath                  TargetPath
----------- ----------------- ----------                  ----------
Data        azurebackup1      F:\Data\azurebackup1.mdf    F:\Data\azurebackup1_1553001753.mdf
Log         azurebackup1_log  F:\Log\azurebackup1_log.ldf F:\Log\azurebackup1_log_1553001753.ldf
```

Définissez les propriétés PowerShell pertinentes en tant que valeurs de chaîne, comme indiqué ci-dessous.

```powershell
$AnotherInstanceWithFullConfig.OverwriteWLIfpresent = "Yes"
$AnotherInstanceWithFullConfig | fl
```

```output
TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : Yes
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
```

> [!IMPORTANT]
> Assurez-vous que l’objet de configuration de récupération final a toutes les valeurs nécessaires et appropriées, puisque l’opération de restauration repose sur l’objet de configuration.

### <a name="restore-with-relevant-configuration"></a>Restaurer avec la configuration appropriée

Une fois que l’objet de configuration de récupération pertinent a été obtenu et vérifié, utilisez l’applet de commande PowerShell [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pour démarrer le processus de restauration.

```powershell
Restore-AzRecoveryServicesBackupItem -WLRecoveryConfig $AnotherInstanceWithLogConfig -VaultId $targetVault.ID
```

L’opération de restauration renvoie un travail à suivre.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Restore              InProgress           3/17/2019 10:02:45 AM                                3274xg2b-e4fg-5952-89b4-8cb566gc1748
```

## <a name="manage-sql-backups"></a>Gérer des sauvegardes SQL

### <a name="on-demand-backup"></a>Sauvegarde à la demande

Une fois la sauvegarde activée pour une base de données, vous pouvez également déclencher une sauvegarde à la demande pour la base de données à l’aide de l’applet de commande PowerShell [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem). L’exemple suivant déclenche une sauvegarde complète sur une base de données SQL avec la compression activée ; la sauvegarde complète doit être conservée pendant 60 jours.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
Backup-AzRecoveryServicesBackupItem -Item $bkpItem -BackupType Full -EnableCompression -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

La commande de sauvegarde à la demande renvoie un travail à suivre.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Backup               InProgress           3/18/2019 8:41:27 PM                                2516bb1a-d3ef-4841-97a3-9ba455fb0637
```

Si la sortie est perdue ou si vous souhaitez obtenir l’ID de travail pertinent, [obtenez la liste des travaux](#track-azure-backup-jobs) à partir du service Sauvegarde Azure, puis suivez-le, ainsi que ses informations.

### <a name="change-policy-for-backup-items"></a>Modifier la stratégie pour les éléments de sauvegarde

Vous pouvez modifier la stratégie de l’élément sauvegardé de *Policy1* en *Policy2*. Pour changer de stratégie pour un élément sauvegardé, récupérez la stratégie et l’élément de sauvegarde appropriés, puis utilisez la commande [Enable-AzRecoveryServices](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) avec l’élément de sauvegarde comme paramètre.

```powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName>
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType MSSQL -BackupManagementType AzureWorkload -Name "<BackupItemName>"
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1
```

La commande attend la fin de la configuration de la sauvegarde et renvoie le résultat suivant.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="edit-an-existing-backup-policy"></a>Modifier une stratégie de sauvegarde existante

Pour modifier une stratégie existante, utilisez la commande [Set-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy).

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -SchedulePolicy $SchPol -RetentionPolicy $RetPol
```

Vérifiez les travaux de sauvegarde au bout d’un certain temps pour suivre l’existence d’échecs éventuels. S’il y en a, vous devez résoudre les problèmes. Réexécutez ensuite la commande de modification de stratégie avec le paramètre **FixForInconsistentItems** pour réessayer de modifier la stratégie sur tous les éléments de sauvegarde pour lesquels l’opération a échoué précédemment.

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -FixForInconsistentItems
```

### <a name="re-register-sql-vms"></a>Inscrire de nouveau des machines virtuelles SQL

> [!WARNING]
> Prenez le temps de lire ce [document](backup-sql-server-azure-troubleshoot.md#re-registration-failures) pour comprendre les symptômes et les causes d’échec avant de tenter la réinscription

Pour déclencher la réinscription de la machine virtuelle SQL, récupérez le conteneur de sauvegarde pertinent et transmettez-le à la cmdlet d’inscription.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Register-AzRecoveryServicesBackupContainer -Container $SQLContainer -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID
```

### <a name="stop-protection"></a>Arrêter la protection

#### <a name="retain-data"></a>Conserver les données

Si vous souhaitez arrêter la protection, vous pouvez utiliser l’applet de commande PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection). Les sauvegardes planifiées seront arrêtées, mais les données sauvegardées jusqu’à présent seront conservées indéfiniment.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
```

#### <a name="delete-backup-data"></a>Supprimer les données de sauvegarde

Pour supprimer complètement les données de sauvegarde stockées dans le coffre, il suffit d’ajouter l’indicateur/l’instruction switch « -RemoveRecoveryPoints » à la [commande de protection « disable »](#retain-data).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
```

#### <a name="disable-auto-protection"></a>Désactiver l’autoprotection

Si l’autoprotection a été configurée sur une SQLInstance, vous pouvez la désactiver à l’aide de l’applet de commande PowerShell [Disable-AzRecoveryServicesBackupAutoProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection).

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Disable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetvault.ID
```

#### <a name="unregister-sql-vm"></a>Désinscrire la machine virtuelle SQL

Si toutes les bases de données d’un serveur SQL [ne sont plus protégées et qu’aucune donnée de sauvegarde n’existe](#delete-backup-data), vous pouvez désinscrire la machine virtuelle SQL de ce coffre. C’est seulement à partir de ce moment-là que vous pouvez protéger des bases de données dans un autre coffre. Utilisez l’applet de commande PowerShell [Unregister-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) pour désinscrire la machine virtuelle SQL.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
 Unregister-AzRecoveryServicesBackupContainer -Container $SQLContainer -VaultId $targetvault.ID
```

### <a name="track-azure-backup-jobs"></a>Suivre les travaux Sauvegarde Azure

Il est important de noter que la sauvegarde Azure suit uniquement les travaux déclenchés par l’utilisateur dans la sauvegarde SQL. Les sauvegardes planifiées (dont les sauvegardes de fichier journal) ne sont pas visibles dans le portail ni dans PowerShell. Toutefois, si un travail planifié échoue, une [alerte de sauvegarde](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault) est générée et affichée dans le portail. [Utilisez Azure Monitor](backup-azure-monitoring-use-azuremonitor.md) pour suivre tous les travaux planifiés et autres informations pertinentes.

Les utilisateurs peuvent suivre des opérations à la demande/déclenchées avec le JobID renvoyé dans la [sortie](#on-demand-backup) des travaux asynchrones comme une sauvegarde. Utilisez l’applet de commande PowerShell [Get-AzRecoveryServicesBackupJobDetail](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjobdetail) pour suivre le travail et ses informations.

```powershell
 Get-AzRecoveryServicesBackupJobDetails -JobId 2516bb1a-d3ef-4841-97a3-9ba455fb0637 -VaultId $targetVault.ID
```

Pour obtenir la liste des travaux à la demande et des états associés auprès du service de sauvegarde Azure, utilisez l’applet de commande PowerShell [Get-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob). L’exemple suivant renvoie tous les travaux SQL en cours d’exécution.

```powershell
Get-AzRecoveryServicesBackupJob -Status InProgress -BackupManagementType AzureWorkload
```

Pour annuler un travail en cours d’exécution, utilisez l’applet de commande PowerShell [Stop-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/stop-azrecoveryservicesbackupjob).

## <a name="managing-sql-always-on-availability-groups"></a>Gestion des groupes de disponibilité SQL AlwaysOn

Pour les groupes de disponibilité SQL AlwaysOnS, veillez à [inscrire tous les nœuds](#registering-the-sql-vm) du groupe de disponibilité. Une fois tous les nœuds inscrits, un objet de groupe de disponibilité SQL est logiquement créé dans les éléments protégeables. Les bases de données sous le groupe de disponibilité SQL seront répertoriées sous la forme « SQLDatabase ». Les nœuds apparaîtront en tant qu’instances autonomes et les bases de données SQL par défaut sous ces derniers seront également répertoriées sous la forme de bases de données SQL.

Supposons par exemple qu’un groupe de disponibilité SQL a deux nœuds, *sql-server-0* et *sql-server-1*, ainsi qu’une base de données de groupe de disponibilité SQL. Une fois que ces deux nœuds sont inscrits, si vous [listez les éléments pouvant être protégés](#fetching-sql-dbs), les composants suivants sont répertoriés

* Un objet de groupe de disponibilité SQL : type d’élément protégeable intitulé SQLAvailabilityGroup
* Une base de données de groupe de disponibilité SQL : type d’élément protégeable intitulé SQLDatabase
* sql-server-0 : type d’élément protégeable intitulé SQLInstance
* sql-server-1 : type d’élément protégeable intitulé SQLInstance
* Les bases de données SQL par défaut (master, modèle, msdb) sous sql-server-0 : type d’élément protégeable intitulé SQLDatabase
* Les bases de données SQL par défaut (master, modèle, msdb) sous sql-server-1 : type d’élément protégeable intitulé SQLDatabase

sql-server-0 et sql-server-1 seront également répertoriés sous la forme « AzureVMAppContainer » dans la [liste des conteneurs de sauvegarde](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).

Récupérez simplement la base de données appropriée pour [activer la sauvegarde](#configuring-backup) et les [applets de commande PowerShell de sauvegarde à la demande](#on-demand-backup) et de [restauration](#restore-sql-dbs) restent identiques.
