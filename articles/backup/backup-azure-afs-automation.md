---
title: Sauvegarder un partage de fichiers Azure en utilisant PowerShell
description: Dans cet article, découvrez comment sauvegarder un partage de fichiers Azure Files avec le service Sauvegarde Azure et PowerShell.
ms.topic: conceptual
ms.date: 08/20/2019
ms.openlocfilehash: 948931764769bc967b88e7942b7e8384b0f93dff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87076997"
---
# <a name="back-up-an-azure-file-share-by-using-powershell"></a>Sauvegarder un partage de fichiers Azure en utilisant PowerShell

Cet article explique comment utiliser Azure PowerShell pour sauvegarder un partage de fichiers Azure Files à l’aide d’un coffre Recovery Services de [Sauvegarde Azure](backup-overview.md).

Cet article explique comment :

> [!div class="checklist"]
>
> * Configurer PowerShell et inscrire le fournisseur Recovery Services.
> * Créez un coffre Recovery Services.
> * Configurer la sauvegarde d’un partage de fichiers Azure.
> * Exécuter un travail de sauvegarde.

## <a name="before-you-start"></a>Avant de commencer

* [En savoir plus](backup-azure-recovery-services-vault-overview.md) sur les coffres Recovery Services.
* Passez en revue la [référence sur la cmdlet](/powershell/module/az.recoveryservices) dans la bibliothèque Azure.
* Passez en revue la hiérarchie des objets PowerShell suivante pour Recovery Services :

  ![Hiérarchie des objets dans Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

## <a name="set-up-powershell"></a>Configurer PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Configurez PowerShell comme suit :

1. [Téléchargez la dernière version d’Azure PowerShell](/powershell/azure/install-az-ps).

    > [!NOTE]
    > La version PowerShell minimale requise pour la sauvegarde du partage de fichiers Azure est Az.RecoveryServices 2.6.0. L’utilisation de la version la plus récente, ou au moins de la version minimale, vous permet d’éviter les problèmes liés aux scripts existants. Installez la version minimale à l’aide de la commande PowerShell suivante :
    >
    > ```powershell
    > Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
    > ```

2. Recherchez les cmdlets PowerShell pour Sauvegarde Azure à l'aide de cette commande :

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Passez en revue les alias et cmdlets pour Sauvegarde Azure, Azure Site Recovery et le coffre Recovery Services. Voici un exemple de résultat possible. Il ne s’agit pas d’une liste complète des cmdlets.

    ![Liste des cmdlets Recovery Services](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Connectez-vous à votre compte Azure à l’aide de **Connect-AzAccount**.
5. Sur la page web qui s’affiche, vous êtes invité à entrer les informations d’identification de votre compte.

    Vous pouvez également inclure les informations d’identification de votre compte en tant que paramètre dans la cmdlet **Connect-AzAccount** à l'aide de **-Credential**.

    Si vous êtes partenaire CSP travaillant pour le compte d’un locataire, spécifiez le client en tant que locataire. Utilisez son ID locataire ou son nom de domaine principal. Par exemple **Connect-AzAccount -Tenant « fabrikam.com »** .

6. Associez l’abonnement que vous souhaitez utiliser avec le compte, car un compte peut compter plusieurs abonnements :

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Si vous utilisez le service Sauvegarde Azure pour la première fois, utilisez la cmdlet **Register-AzResourceProvider** afin d'associer le fournisseur Azure Recovery Service à votre abonnement :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Vérifiez que les fournisseurs ont été correctement inscrits :

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. Dans la sortie de commande, vérifiez que **RegistrationState** a maintenant la valeur **Inscrit**. Si ce n’est pas ce cas, réexécutez la cmdlet **Register-AzResourceProvider**.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Le coffre Recovery Services étant une ressource Resource Manager, vous devez le placer dans un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un à l'aide de la cmdlet **New-AzResourceGroup**. Lorsque vous créez un groupe de ressources, spécifiez son nom et son emplacement.

Pour créer un coffre Recovery Services, procédez comme suit :

1. Si vous n’avez pas de groupe de ressources, créez-en un à l'aide de la cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Dans l’exemple suivant, nous créons un groupe de ressources dans la région USA Ouest.

   ```powershell
   New-AzResourceGroup -Name "test-rg" -Location "West US"
   ```

1. Utilisez la cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) pour créer le coffre. Pour le coffre, spécifiez le même emplacement que celui utilisé pour le groupe de ressources.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Afficher les coffres dans un abonnement

Pour afficher tous les coffres de l'abonnement, utilisez [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault) :

```powershell
Get-AzRecoveryServicesVault
```

Le résultat ressemble à ce qui suit. Notez que la sortie fournit le groupe de ressources et l’emplacement associés.

```powershell
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

Étant donné que de nombreuses cmdlets Sauvegarde Azure exigent l’objet de coffre Recovery Services en tant qu’entrée, il est judicieux de le stocker dans une variable.

Le contexte du coffre spécifie le type de données protégées dans le coffre. Définissez-le à l'aide de [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). Une fois le contexte défini, il s’applique à toutes les cmdlets suivantes.

L’exemple suivant définit le contexte pour le coffre **testvault** :

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Récupérer l’ID de coffre

Conformément aux instructions d’Azure PowerShell, nous envisageons de déprécier la définition du contexte de coffre. À la place, vous pouvez stocker ou récupérer l’ID du coffre et le transmettre aux commandes appropriées. Si vous n’avez pas défini le contexte du coffre ou si vous souhaitez spécifier la commande à exécuter pour un coffre donné, transmettez l’ID du coffre sous la forme `-vaultID` dans toutes les commandes concernées :

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultID $vaultID
```

## <a name="configure-a-backup-policy"></a>Configurer une stratégie de sauvegarde

Une stratégie de sauvegarde spécifie la planification des sauvegardes et la durée de conservation des points de récupération de sauvegarde.

Une stratégie de sauvegarde est associée à au moins une stratégie de rétention. Une stratégie de rétention définit la durée de conservation d’un point de restauration avant sa suppression. Vous pouvez configurer des sauvegardes avec une rétention quotidienne, hebdomadaire, mensuelle ou annuelle.

Voici quelques cmdlets pour les stratégies de sauvegarde :

* Affichez la stratégie de rétention de sauvegarde par défaut à l'aide de [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Affichez la planification de la stratégie de sauvegarde par défaut à l'aide de [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* Créez une stratégie de sauvegarde à l'aide de [New-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy). Vous entrez les objets de stratégie de planification et de rétention.

Par défaut, une heure de début est définie dans l’objet de la stratégie de planification. Utilisez l’exemple suivant pour modifier l’heure de début sur l’heure de début souhaitée. L’heure de début souhaitée doit être au format UTC (Universal Coordinated Time). L’exemple suppose que l’heure de début souhaitée est 01:00 AM UTC pour les sauvegardes quotidiennes.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Vous devez fournir l’heure de début en multiples de 30 minutes uniquement. Dans l’exemple précédent, il peut s'agir uniquement de « 01:00:00 » ou de « 02:30:00 ». L’heure de début ne peut pas être « 01:15:00 ».

L’exemple suivant stocke la stratégie de planification et la stratégie de conservation dans des variables. Il utilise ensuite ces variables en tant que paramètres pour une nouvelle stratégie (**NewAFSPolicy**). La stratégie **NewAFSPolicy** effectue une sauvegarde quotidienne et la conserve pendant 30 jours.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureFiles"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

Le résultat ressemble à ce qui suit :

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewAFSPolicy           AzureFiles            AzureStorage              10/24/2019 1:30:00 AM
```

## <a name="enable-backup"></a>Activer la sauvegarde

Après avoir défini la stratégie de sauvegarde, vous pouvez l’utiliser pour activer la protection du partage de fichiers Azure.

### <a name="retrieve-a-backup-policy"></a>Récupérer une stratégie de sauvegarde

Récupérez l’objet de stratégie approprié à l'aide de [Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy). Utilisez cette cmdlet pour afficher les stratégies associées à un type de charge de travail ou pour obtenir une stratégie spécifique.

#### <a name="retrieve-a-policy-for-a-workload-type"></a>Récupérer une stratégie pour un type de charge de travail

L’exemple suivant permet de récupérer des stratégies pour le type de charge de travail **AzureFiles** :

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureFiles"
```

Le résultat ressemble à ce qui suit :

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
dailyafs             AzureFiles         AzureStorage         1/10/2018 12:30:00 AM
```

> [!NOTE]
> Le fuseau horaire du champ **BackupTime** dans PowerShell se présente au format UTC. Lorsque l’heure de sauvegarde s’affiche dans le portail Azure, elle est alignée sur votre fuseau horaire.

#### <a name="retrieve-a-specific-policy"></a>Récupérer une stratégie spécifique

La stratégie suivante récupère la stratégie de sauvegarde intitulée **dailyafs** :

```powershell
$afsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "dailyafs"
```

### <a name="enable-protection-and-apply-the-policy"></a>Activer la protection et appliquer la stratégie

Activez la protection à l'aide de [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Une fois la stratégie associée au coffre, les sauvegardes sont déclenchées conformément à la planification de la stratégie.

L’exemple suivant active la protection du partage de fichiers Azure **testAzureFileShare** dans le compte de stockage **testStorageAcct**, avec la stratégie **dailyafs** :

```powershell
Enable-AzRecoveryServicesBackupProtection -StorageAccountName "testStorageAcct" -Name "testAzureFS" -Policy $afsPol
```

La commande attend que le travail de configuration de la protection soit terminé et produit une sortie similaire à celle illustrée dans l'exemple suivant :

```cmd
WorkloadName       Operation            Status                 StartTime                                                                                                         EndTime                   JobID
------------             ---------            ------               ---------                                  -------                   -----
testAzureFS       ConfigureBackup      Completed            11/12/2018 2:15:26 PM     11/12/2018 2:16:11 PM     ec7d4f1d-40bd-46a4-9edb-3193c41f6bf6
```

Pour plus d’informations sur l’obtention d’une liste de partages de fichiers pour un compte de stockage, consultez [cet article](/powershell/module/az.storage/get-azstorageshare).

## <a name="important-notice-backup-item-identification"></a>Remarque importante : Identification de l’élément de sauvegarde

Cette section décrit un changement important en termes de sauvegardes de partages de fichiers Azure pour préparer la mise à la disposition générale.

Lors de l’activation d'une sauvegarde pour partages de fichiers Azure, l’utilisateur fournit un nom de partage de fichiers du client comme nom d’entité ; un élément de sauvegarde est alors créé. Le nom de l’élément de sauvegarde est un identificateur unique créé par le service Sauvegarde Azure. En général, l’identificateur correspond à un nom convivial. Cependant, pour gérer le scénario des suppressions réversibles, où un partage de fichiers peut être supprimé et un autre partage de fichiers créé avec le même nom, l’identité unique d'un partage de fichiers Azure correspond désormais à un ID.

Pour connaître l’ID unique de chaque élément, exécutez la commande **Get-AzRecoveryServicesBackupItem** avec les filtres appropriés pour **backupManagementType** et **WorkloadType** afin d'obtenir tous les éléments pertinents. Observez ensuite le champ de nom dans la réponse/l’objet PowerShell retourné.

Nous vous recommandons de lister les éléments, puis de récupérer leur nom unique dans le champ de nom de la réponse. Utilisez cette valeur pour filtrer les éléments avec le paramètre *Name*. Sinon, utilisez le paramètre *FriendlyName* pour récupérer l’élément avec son ID.

> [!IMPORTANT]
> Assurez-vous que PowerShell est mis à niveau vers la version minimale (Az. RecoveryServices 2.6.0) pour les sauvegardes des partages de fichiers Azure. Avec cette version, le filtre *FriendlyName* est disponible pour la commande **Get-AzRecoveryServicesBackupItem**.
>
> Passez le nom du partage de fichiers Azure au paramètre *FriendlyName*. Si vous passez le nom du partage de fichiers Azure au paramètre *Name*, cette version génère un avertissement indiquant de passer ce nom au paramètre *FriendlyName*.
>
> Si vous n’installez pas cette version minimale, vous risquez de provoquer un échec des scripts existants. Installez la version minimale de PowerShell à l'aide de la commande suivante :
>
>```powershell
>Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
>```

## <a name="trigger-an-on-demand-backup"></a>Déclencher une sauvegarde à la demande

Utilisez [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) pour exécuter une sauvegarde à la demande pour un partage de fichiers Azure protégé :

1. Récupérez le compte de stockage auprès du conteneur dans le coffre qui contient vos données de sauvegarde à l'aide de [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Pour démarrer un travail de sauvegarde, obtenez des informations sur le partage de fichiers Azure à l'aide de [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Exécutez une sauvegarde à la demande à l'aide de [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).

Exécutez la sauvegarde à la demande comme suit :

```powershell
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType "AzureFiles" -FriendlyName "testAzureFS"
$job =  Backup-AzRecoveryServicesBackupItem -Item $afsBkpItem
```

La commande retourne un travail avec un ID à suivre, comme indiqué dans l’exemple suivant :

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS       Backup               Completed            11/12/2018 2:42:07 PM     11/12/2018 2:42:11 PM     8bdfe3ab-9bf7-4be6-83d6-37ff1ca13ab6
```

Les instantanés de partage de fichiers Azure sont utilisés lors des sauvegardes. Le travail est généralement terminé au moment où la commande retourne cette sortie.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur la [sauvegarde Azure Files dans le portail Azure](backup-afs.md).
* Reportez-vous à l’[exemple de script sur GitHub](https://github.com/Azure-Samples/Use-PowerShell-for-long-term-retention-of-Azure-Files-Backup) pour utiliser un runbook Azure Automation afin de planifier des sauvegardes.
