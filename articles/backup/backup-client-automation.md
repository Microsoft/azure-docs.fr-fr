---
title: Utiliser PowerShell pour sauvegarder Windows Server dans Azure
description: Dans cet article, découvrez comment utiliser PowerShell pour configurer Sauvegarde Azure sur Windows Server ou sur un client Windows, ainsi que pour gérer les sauvegardes et la récupération.
ms.topic: conceptual
ms.date: 12/2/2019
ms.openlocfilehash: 582d8123f16b2d5a543d862b8eb3e45895087e4a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987094"
---
# <a name="deploy-and-manage-backup-to-azure-for-windows-serverwindows-client-using-powershell"></a>Déployer et gérer une sauvegarde vers Azure pour un serveur/client Windows à l’aide de PowerShell

Cet article explique comment utiliser PowerShell pour configurer Sauvegarde Azure sur Windows Server ou sur un client Windows et comment gérer les sauvegardes et la récupération.

## <a name="install-azure-powershell"></a>Installation d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour commencer, [installez la dernière version de PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Les étapes suivantes vous montrent comment créer un coffre Recovery Services. Un coffre Recovery Services diffère d’un coffre de sauvegarde.

1. Si vous utilisez Sauvegarde Azure pour la première fois, vous devez utiliser la cmdlet **Register-AzResourceProvider** pour associer le fournisseur Azure Recovery Service à votre abonnement.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

2. Le coffre Recovery Services constituant une ressource Azure Resource Manager, vous devez le placer dans un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un. Quand vous créez un groupe de ressources, spécifiez ses nom et emplacement.  

    ```powershell
    New-AzResourceGroup –Name "test-rg" –Location "WestUS"
    ```

3. Utilisez l’applet de commande **New-AzRecoveryServicesVault** pour créer le coffre. Spécifiez pour le coffre le même emplacement que pour le groupe de ressources.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "WestUS"
    ```

4. Spécifiez le type de redondance de stockage à utiliser. Vous pouvez utiliser le [stockage localement redondant (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage), le [stockage géoredondant (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage) ou le [stockage redondant interzone (ZRS)](../storage/common/storage-redundancy.md#zone-redundant-storage). L'exemple suivant montre que l'option **-BackupStorageRedundancy** de *testVault* est définie sur **GeoRedundant**.

   > [!TIP]
   > De nombreuses applets de commande Azure Backup nécessitent l’objet coffre Recovery Services en tant qu’entrée. Il est donc pratique de stocker l'objet coffre Backup Recovery Services dans une variable.
   >
   >

    ```powershell
    $Vault1 = Get-AzRecoveryServicesVault –Name "testVault"
    Set-AzRecoveryServicesBackupProperties -Vault $Vault1 -BackupStorageRedundancy GeoRedundant
    ```

## <a name="view-the-vaults-in-a-subscription"></a>Afficher les coffres dans un abonnement

Utilisez **Get-AzRecoveryServicesVault** pour voir la liste de tous les coffres dans l’abonnement actuel. Vous pouvez utiliser cette commande pour vérifier qu’un coffre a été créé ou pour voir quels coffres sont disponibles dans l’abonnement.

Exécutez la commande **Get-AzRecoveryServicesVault** ; tous les coffres de l’abonnement sont alors listés.

```powershell
Get-AzRecoveryServicesVault
```

```Output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="installing-the-azure-backup-agent"></a>Installation de l'agent de sauvegarde Azure

Avant d’installer l'agent de sauvegarde Azure, vous devez avoir téléchargé le programme d’installation sur le serveur Windows. Vous pouvez obtenir la dernière version du programme d’installation à partir du [Centre de téléchargement Microsoft](https://aka.ms/azurebackup_agent) ou de la page Tableau de bord du coffre Recovery Services. Enregistrez le programme d’installation à un emplacement auquel vous pouvez accéder facilement, par exemple `C:\Downloads\*`.

Vous pouvez également utiliser PowerShell pour obtenir le téléchargeur :

 ```powershell
 $MarsAURL = 'https://aka.ms/Azurebackup_Agent'
 $WC = New-Object System.Net.WebClient
 $WC.DownloadFile($MarsAURL,'C:\downloads\MARSAgentInstaller.EXE')
 C:\Downloads\MARSAgentInstaller.EXE /q
 ```

Pour installer l’agent, exécutez la commande ci-après dans une console PowerShell avec élévation de privilèges :

```powershell
MARSAgentInstaller.exe /q
```

Cette opération installe l’agent avec les options par défaut. L’installation s’effectue en arrière-plan et prend quelques minutes. Si vous ne spécifiez pas l’option */nu*, la fenêtre **Windows Update** s’ouvrira à la fin de l’installation pour rechercher des mises à jour. Une fois installé, l’agent apparaît dans la liste des programmes installés.

Pour afficher la liste des programmes installés, cliquez sur **Panneau de configuration** > **Programmes** > **Programmes et fonctionnalités**.

![Agent installé](./media/backup-client-automation/installed-agent-listing.png)

### <a name="installation-options"></a>Options d'installation

Pour afficher toutes les options disponibles via la ligne de commande, utilisez la commande suivante :

```powershell
MARSAgentInstaller.exe /?
```

Les options disponibles incluent :

| Option | Détails | Default |
| --- | --- | --- |
| /q |Installation silencieuse |- |
| /p:"emplacement" |Chemin d’accès du dossier d’installation de l’agent de Sauvegarde Azure. |C:\Program Files\Microsoft Azure Recovery Services Agent |
| /s:"emplacement" |Chemin d’accès du dossier de cache de l’agent de Sauvegarde Azure. |C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch |
| /m |Abonnement à Microsoft Update |- |
| /nu |Ne pas vérifier les mises à jour une fois l’installation terminée |- |
| /d |Désinstalle l’agent Microsoft Azure Recovery Services |- |
| /ph |Adresse de l’hôte proxy |- |
| /po |Numéro de port de l’hôte proxy |- |
| /pu |Nom d’utilisateur de l’hôte proxy |- |
| /pw |Mot de passe du proxy |- |

## <a name="registering-windows-server-or-windows-client-machine-to-a-recovery-services-vault"></a>Inscription d'un serveur Windows Server ou d'un ordinateur client Windows auprès d'un coffre Recovery Services

Une fois que vous avez créé un coffre Recovery Services, téléchargez le dernier agent et les informations d’identification de coffre et stockez-les dans un emplacement pratique comme C:\Téléchargements.

```powershell
$CredsPath = "C:\downloads"
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault1 -Path $CredsPath
```

### <a name="registering-using-the-ps-az-module"></a>Inscription à l’aide du module PS Az

> [!NOTE]
> Un bogue lié à la génération de certificat de coffre a été résolu dans la version Az 3.5.0. Utilisez la version Az 3.5.0 ou une version ultérieure pour télécharger un certificat de coffre.

Dans le dernier module Az de PowerShell, en raison des limitations sous-jacentes de la plateforme, le téléchargement des informations d’identification du coffre nécessite un certificat autosigné. L’exemple suivant montre comment fournir un certificat auto-signé et télécharger les informations d’identification du coffre-fort.

```powershell
$dt = $(Get-Date).ToString("M-d-yyyy")
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -FriendlyName 'test-vaultcredentials' -subject "Windows Azure Tools" -KeyExportPolicy Exportable -NotAfter $(Get-Date).AddHours(48) -NotBefore $(Get-Date).AddHours(-24) -KeyProtection None -KeyUsage None -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2") -Provider "Microsoft Enhanced Cryptographic Provider v1.0"
$certficate = [convert]::ToBase64String($cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx))
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault -Path $CredsPath -Certificate $certficate
```

Sur le serveur Windows Server ou l’ordinateur client Windows, exécutez l’applet de commande [Start-OBRegistration](/powershell/module/msonlinebackup/start-obregistration) pour inscrire l’ordinateur auprès du coffre.
Cette cmdlet, ainsi que d’autres utilisées pour la sauvegarde, proviennent du module MSONLINE que l’AgentInstaller MARS a ajouté dans le cadre du processus d’installation.

Le programme d’installation de l’agent ne met pas à jour la variable $Env:PSModulePath. Cela signifie que le chargement automatique du module échoue. Pour résoudre ce problème, vous pouvez effectuer les étapes suivantes :

```powershell
$Env:PSModulePath += ';C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules'
```

En guise d’alternative, vous pouvez charger manuellement le module dans votre script comme suit :

```powershell
Import-Module -Name 'C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup'
```

Une fois que vous avez chargé les applets de commande de Sauvegarde en ligne, vous devez inscrire les informations d’identification du coffre :

```powershell
Start-OBRegistration -VaultCredentials $CredsFilename.FilePath -Confirm:$false
```

```Output
CertThumbprint      : 7a2ef2caa2e74b6ed1222a5e89288ddad438df2
SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
ServiceResourceName : testvault
Region              : WestUS
Machine registration succeeded.
```

> [!IMPORTANT]
> N'utilisez pas de chemins d'accès relatifs pour spécifier le fichier des informations d'identification du coffre. Vous devez fournir un chemin absolu dans l’applet de commande.
>
>

## <a name="networking-settings"></a>Paramètres de mise en réseau

Lorsque l’ordinateur Windows accède à Internet via un serveur proxy, les paramètres proxy peuvent également être fournis à l’agent. Dans cet exemple, il n’y a aucun serveur proxy. Nous effaçons donc explicitement toutes les informations relatives au proxy.

L’utilisation de la bande passante peut également être contrôlée avec les options `work hour bandwidth` et `non-work hour bandwidth`, certains jours de la semaine.

La définition des détails sur le proxy et la bande passante s’effectue à l’aide de l’applet de commande [Set-OBMachineSetting](/powershell/module/msonlinebackup/set-obmachinesetting) :

```powershell
Set-OBMachineSetting -NoProxy
```

```Output
Server properties updated successfully.
```

```powershell
Set-OBMachineSetting -NoThrottle
```

```Output
Server properties updated successfully.
```

## <a name="encryption-settings"></a>Paramètres de chiffrement

Les données sauvegardées envoyées à Sauvegarde Azure sont chiffrées pour garantir leur confidentialité. Le mot de passe du chiffrement est le « mot de passe » permettant de déchiffrer les données lors de la restauration.

Vous devez générer un code pin de sécurité en sélectionnant **Générer**, sous **Paramètres** > **Propriétés** > **Code PIN de sécurité** dans la section **Coffre Recovery Services** du portail Azure.

>[!NOTE]
> Le code PIN de sécurité ne peut être généré que par l’intermédiaire du Portail Azure.

Ensuite, utilisez-le comme `generatedPIN` dans la commande :

```powershell
$PassPhrase = ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force
Set-OBMachineSetting -EncryptionPassPhrase $PassPhrase -SecurityPin "<generatedPIN>"
```

```Output
Server properties updated successfully
```

> [!IMPORTANT]
> Conservez les informations relatives à la phrase secrète en lieu sûr après les avoir définies. Vous ne pouvez pas restaurer les données à partir d’Azure sans cette phrase secrète.
>
>

## <a name="back-up-files-and-folders"></a>Sauvegarde des fichiers et dossiers

Toutes les sauvegardes de serveurs et clients Windows vers Sauvegarde Azure sont régies par une stratégie. Cette dernière comprend trois parties :

1. Une **planification de sauvegarde** qui spécifie quand les sauvegardes doivent être établies et synchronisées avec le service.
2. Une **planification de rétention** qui spécifie la durée de rétention des points de récupération dans Azure.
3. Une **spécification d'inclusion/exclusion de fichier** qui dicte ce qui doit être sauvegardé.

Dans ce document, comme nous automatisons la sauvegarde, nous supposons que rien n'a été configuré. Nous commençons par créer une stratégie de sauvegarde en utilisant l’applet de commande [New-OBPolicy](/powershell/module/msonlinebackup/new-obpolicy) .

```powershell
$NewPolicy = New-OBPolicy
```

À ce stade, la stratégie est vide et d’autres cmdlets sont nécessaires pour définir quels éléments seront inclus ou exclus, quand les sauvegardes seront exécutées et où celles-ci seront stockées.

### <a name="configuring-the-backup-schedule"></a>Configuration de la planification de sauvegarde

La première des trois parties d’une stratégie est la planification de sauvegarde, qui est créée à l’aide de l’applet de commande [New-OBSchedule](/powershell/module/msonlinebackup/new-obschedule). La planification de sauvegarde définit le moment où les sauvegardes doivent être effectuées. Lors de la création d’une planification, vous devez spécifier deux paramètres d’entrée :

* **jours de la semaine** où la sauvegarde doit s'exécuter. Vous pouvez exécuter le travail de sauvegarde une seule journée ou tous les jours de la semaine, ou une combinaison des deux.
* **heures** où la sauvegarde doit être exécutée. Vous pouvez définir jusqu’à trois différentes heures pour le déclenchement de la sauvegarde.

Par exemple, vous pouvez configurer une stratégie de sauvegarde qui s'exécute à 16 h 00 chaque samedi et chaque dimanche.

```powershell
$Schedule = New-OBSchedule -DaysOfWeek Saturday, Sunday -TimesOfDay 16:00
```

La planification de sauvegarde doit être associée à une stratégie à l'aide de l’applet de commande [Set-OBSchedule](/powershell/module/msonlinebackup/set-obschedule) .

```powershell
Set-OBSchedule -Policy $NewPolicy -Schedule $Schedule
```

```Output
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid
```

### <a name="configuring-a-retention-policy"></a>Configuration d'une stratégie de rétention

La stratégie de rétention définit la durée de conservation des points de récupération créés à partir des travaux de sauvegarde. Lorsque vous créez une stratégie de rétention à l’aide de la cmdlet [New-OBRetentionPolicy](/powershell/module/msonlinebackup/new-obretentionpolicy), vous pouvez spécifier le nombre de jours pendant lesquels les points de récupération de sauvegarde doivent être conservés avec Sauvegarde Azure. L’exemple suivant définit une stratégie de rétention de sept jours.

```powershell
$RetentionPolicy = New-OBRetentionPolicy -RetentionDays 7
```

La stratégie de rétention doit être associée à la stratégie principale à l'aide de l'applet de commande [Set-OBRetentionPolicy](/powershell/module/msonlinebackup/set-obretentionpolicy):

```powershell
Set-OBRetentionPolicy -Policy $NewPolicy -RetentionPolicy $RetentionPolicy
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          :
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="including-and-excluding-files-to-be-backed-up"></a>Inclusion et exclusion des fichiers à sauvegarder

Un objet `OBFileSpec` définit les fichiers à inclure et à exclure d'une sauvegarde. Il s'agit d'un ensemble de règles qui définissent l'étendue des fichiers et dossiers protégés sur un ordinateur. Vous pouvez avoir de nombreuses règles d'inclusion ou d’exclusion en fonction de vos besoins, et les associer à une stratégie. Lorsque vous créez un objet OBFileSpec, vous pouvez :

* Spécifier les fichiers et dossiers à inclure
* Spécifier les fichiers et dossiers à exclure
* Indiquer la sauvegarde récursive des données dans un dossier (ou) indiquer si seuls les fichiers de niveau supérieur du dossier spécifié doivent être sauvegardés

Dans le dernier cas, l’opération est effectuée à l’aide de l'indicateur -NonRecursive dans la commande New-OBFileSpec.

Dans l'exemple ci-dessous, nous sauvegardons les volumes C: et D: et excluons les fichiers binaires de système d'exploitation dans le dossier Windows et tous les dossiers temporaires. Pour cela, nous allons créer deux spécifications de fichiers à l’aide de l’applet de commande [New-OBFileSpec](/powershell/module/msonlinebackup/new-obfilespec) : une pour l’inclusion et une pour l’exclusion. Une fois que les spécifications de fichiers ont été créées, elles sont associées à la stratégie à l'aide de l’applet de commande [Add-OBFileSpec](/powershell/module/msonlinebackup/add-obfilespec) .

```powershell
$Inclusions = New-OBFileSpec -FileSpec @("C:\", "D:\")
$Exclusions = New-OBFileSpec -FileSpec @("C:\windows", "C:\temp") -Exclude
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Inclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

```powershell
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Exclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\windows
                  IsExclude:True
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\temp
                  IsExclude:True
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="applying-the-policy"></a>Application de la stratégie

L'objet de stratégie est à présent complet. Il est associé à une planification de sauvegarde, à une stratégie de rétention et à une liste d’inclusion/exclusion de fichiers. Cette stratégie peut maintenant être validée à des fins d’utilisation par Sauvegarde Azure. Avant d’appliquer la stratégie que vous venez de créer, vérifiez qu’aucune stratégie de sauvegarde existante n’est associée au serveur à l’aide de l’applet de commande [Remove-OBPolicy](/powershell/module/msonlinebackup/remove-obpolicy). Lors de la suppression de la stratégie, vous êtes invité à confirmer l'opération. Pour ignorer la confirmation, utilisez l’indicateur `-Confirm:$false` avec l’applet de commande.

```powershell
Get-OBPolicy | Remove-OBPolicy
```

```Output
Microsoft Azure Backup Are you sure you want to remove this backup policy? This will delete all the backed up data. [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
```

La validation de l'objet de stratégie s'effectue à l'aide de l’applet de commande [Set-OBPolicy](/powershell/module/msonlinebackup/set-obpolicy) . Une confirmation vous est également demandée. Pour ignorer la confirmation, utilisez l’indicateur `-Confirm:$false` avec l’applet de commande.

```powershell
Set-OBPolicy -Policy $NewPolicy
```

```Output
Microsoft Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s)
DsList : {DataSource
         DatasourceId:4508156004108672185
         Name:C:\
         FileSpec:FileSpec
         FileSpec:C:\
         IsExclude:False
         IsRecursive:True,

         FileSpec
         FileSpec:C:\windows
         IsExclude:True
         IsRecursive:True,

         FileSpec
         FileSpec:C:\temp
         IsExclude:True
         IsRecursive:True,

         DataSource
         DatasourceId:4508156005178868542
         Name:D:\
         FileSpec:FileSpec
         FileSpec:D:\
         IsExclude:False
         IsRecursive:True
    }
PolicyName : c2eb6568-8a06-49f4-a20e-3019ae411bac
RetentionPolicy : Retention Days : 7
              WeeklyLTRSchedule :
              Weekly schedule is not set

              MonthlyLTRSchedule :
              Monthly schedule is not set

              YearlyLTRSchedule :
              Yearly schedule is not set
State : Existing PolicyState : Valid
```

Vous pouvez afficher les détails de la stratégie de sauvegarde existante à l'aide de l’applet de commande [Get-OBPolicy](/powershell/module/msonlinebackup/get-obpolicy) . Vous pouvez afficher plus de détails à l’aide de l’applet de commande [Get-OBSchedule](/powershell/module/msonlinebackup/get-obschedule) pour la planification de sauvegarde et de l’applet de commande [Get-OBRetentionPolicy](/powershell/module/msonlinebackup/get-obretentionpolicy) pour les stratégies de rétention

```powershell
Get-OBPolicy | Get-OBSchedule
```

```Output
SchedulePolicyName : 71944081-9950-4f7e-841d-32f0a0a1359a
ScheduleRunDays : {Saturday, Sunday}
ScheduleRunTimes : {16:00:00}
State : Existing
```

```powershell
Get-OBPolicy | Get-OBRetentionPolicy
```

```Output
RetentionDays : 7
RetentionPolicyName : ca3574ec-8331-46fd-a605-c01743a5265e
State : Existing
```

```powershell
Get-OBPolicy | Get-OBFileSpec
```

```Output
FileName : *
FilePath : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
FileSpec : D:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\
FileSpec : C:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\windows
FileSpec : C:\windows
IsExclude : True
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\temp
FileSpec : C:\temp
IsExclude : True
IsRecursive : True
```

### <a name="performing-an-on-demand-backup"></a>Effectuer une sauvegarde à la demande

Une fois qu’une stratégie de sauvegarde a été définie, les sauvegardes ont lieu selon la planification indiquée. Il est également possible de déclencher une sauvegarde à la demande avec la cmdlet [Start-OBBackup](/powershell/module/msonlinebackup/start-obbackup) :

```powershell
Get-OBPolicy | Start-OBBackup
```

```Output
Initializing
Taking snapshot of volumes...
Preparing storage...
Generating backup metadata information and preparing the metadata VHD...
Data transfer is in progress. It might take longer since it is the first backup and all data needs to be transferred...
Data transfer completed and all backed up data is in the cloud. Verifying data integrity...
Data transfer completed
In progress...
Job completed.
The backup operation completed successfully.
```

## <a name="back-up-windows-server-system-state-in-mars-agent"></a>Sauvegarder l’état du système Windows Server dans l’agent MARS

Cette section traite de la commande PowerShell permettant de configurer l’état du système dans l’agent MARS.

### <a name="schedule"></a>Planifier

```powershell
$sched = New-OBSchedule -DaysOfWeek Sunday,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday -TimesOfDay 2:00
```

### <a name="retention"></a>Rétention

```powershell
$rtn = New-OBRetentionPolicy -RetentionDays 32 -RetentionWeeklyPolicy -RetentionWeeks 13 -WeekDaysOfWeek Sunday -WeekTimesOfDay 2:00  -RetentionMonthlyPolicy -RetentionMonths 13 -MonthDaysOfMonth 1 -MonthTimesOfDay 2:00
```

### <a name="configuring-schedule-and-retention"></a>Configuration de la planification et de la conservation

```powershell
New-OBPolicy | Add-OBSystemState |  Set-OBRetentionPolicy -RetentionPolicy $rtn | Set-OBSchedule -Schedule $sched | Set-OBSystemStatePolicy
 ```

### <a name="verifying-the-policy"></a>Vérification de la stratégie

```powershell
Get-OBSystemStatePolicy
 ```

## <a name="restore-data-from-azure-backup"></a>Restauration des données à partir de Sauvegarde Azure

Cette section vous guide tout au long des étapes d'automatisation de la récupération des données à partir de Sauvegarde Azure. Cette opération implique les étapes suivantes :

1. Sélection du volume source
2. Choix d’un point de sauvegarde à restaurer
3. Spécifier un élément à restaurer
4. Déclenchement du processus de restauration

### <a name="picking-the-source-volume"></a>Sélection du volume source

Pour restaurer un élément à partir de Sauvegarde Azure, vous devez d’abord identifier la source associée. Étant donné que nous exécutons les commandes dans le contexte d'un serveur ou d’un client Windows, l'ordinateur est déjà identifié. L'étape suivante pour identifier la source consiste à identifier le volume qui la contient. Vous pouvez récupérer la liste des volumes ou des sources en cours de sauvegarde à partir de cet ordinateur en exécutant l’applet de commande [Get-OBRecoverableSource](/powershell/module/msonlinebackup/get-obrecoverablesource) . Cette commande renvoie un tableau de toutes les sources sauvegardées à partir de ce serveur/client.

```powershell
$Source = Get-OBRecoverableSource
$Source
```

```Output
FriendlyName : C:\
RecoverySourceName : C:\
ServerName : myserver.microsoft.com

FriendlyName : D:\
RecoverySourceName : D:\
ServerName : myserver.microsoft.com
```

### <a name="choosing-a-backup-point-from-which-to-restore"></a>Choix d’un point de sauvegarde à partir duquel restaurer

Vous récupérez une liste des points de sauvegarde en exécutant l’applet de commande [Get-OBRecoverableItem](/powershell/module/msonlinebackup/get-obrecoverableitem) avec les paramètres appropriés. Dans notre exemple, nous allons sélectionner le dernier point de sauvegarde du volume source *C:* et l’utiliser pour récupérer un fichier spécifique.

```powershell
$Rps = Get-OBRecoverableItem $Source[0]
$Rps
```

```Output

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/16/2019 7:00:19 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :
```

L'objet `$Rps` est un tableau de points de sauvegarde. Le premier élément est le point le plus récent et le Nième élément est le point le plus ancien. Pour choisir le point le plus récent, nous allons utiliser `$Rps[0]`.

### <a name="specifying-an-item-to-restore"></a>Spécification d’un élément à restaurer

Pour restaurer un fichier spécifique, spécifiez son nom par rapport au volume racine. Par exemple, pour récupérer C:\Test\Cat.job, exécutez la commande suivante.

```powershell
$Item = New-OBRecoverableItem $Rps[0] "Test\cat.jpg" $FALSE
$Item
```

```Output
IsDir                : False
ItemNameFriendly     : C:\Test\cat.jpg
ItemNameGuid         :
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : cat.jpg
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime : 21-Jun-14 6:43:02 AM

```

### <a name="triggering-the-restore-process"></a>Déclenchement du processus de restauration

Pour déclencher le processus de restauration, nous devons d'abord spécifier les options de récupération. Pour ce faire, utilisez l’applet de commande [New-OBRecoveryOption](/powershell/module/msonlinebackup/new-obrecoveryoption) . Dans le cadre de cet exemple, supposons que vous souhaitez restaurer les fichiers dans *C:\temp*. Supposons également que vous souhaitez ignorer les fichiers qui existent déjà dans le dossier de destination *C:\temp*. Pour créer une telle option de récupération, utilisez la commande suivante :

```powershell
$RecoveryOption = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip
```

Déclenchez à présent le processus de restauration en exécutant la commande [Start-OBRecovery](/powershell/module/msonlinebackup/start-obrecovery) sur l’élément `$Item` sélectionné à partir de la sortie de l’applet de commande `Get-OBRecoverableItem` :

```powershell
Start-OBRecovery -RecoverableItem $Item -RecoveryOption $RecoveryOption
```

```Output
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Job completed.
The recovery operation completed successfully.
```

## <a name="uninstalling-the-azure-backup-agent"></a>Désinstallation de l’agent de sauvegarde Azure

La désinstallation de l’agent de sauvegarde Azure peut être effectuée à l’aide de la commande suivante :

```powershell
.\MARSAgentInstaller.exe /d /q
```

La désinstallation des fichiers binaires de l'agent de l'ordinateur a certaines conséquences à prendre en compte :

* Elle supprime le filtre de fichier de l'ordinateur et le suivi des modifications est arrêté.
* Toutes les informations de stratégie sont supprimées de l'ordinateur, mais elles continuent à être stockées dans le service.
* Toutes les planifications de sauvegarde sont supprimées, et aucune autre sauvegarde n'est effectuée.

Cependant, les données stockées dans Azure sont conservées conformément à la stratégie de conservation que vous avez définie. Les points plus anciens deviennent automatiquement obsolètes.

## <a name="remote-management"></a>Gestion à distance

L’intégralité de la gestion concernant l’agent de sauvegarde Azure, les stratégies et les sources de données peut être effectuée à distance par le biais de PowerShell. L’ordinateur qui sera géré à distance doit être correctement préparé.

Par défaut, le service WinRM est configuré pour un démarrage manuel. Le type de démarrage doit être défini sur *Automatique* et le service devrait être démarré. Pour vérifier que le service WinRM est exécuté, la valeur de la propriété État devrait être défini sur *En cours d’exécution*.

```powershell
Get-Service -Name WinRM
```

```Output
Status   Name               DisplayName
------   ----               -----------
Running  winrm              Windows Remote Management (WS-Manag...
```

PowerShell doit être configuré pour la communication à distance.

```powershell
Enable-PSRemoting -Force
```

```Output
WinRM is already set up to receive requests on this computer.
WinRM has been updated for remote management.
WinRM firewall exception enabled.
```

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force
```

L’ordinateur peut maintenant être géré à distance, en commençant par l’installation de l’agent. Par exemple, le script suivant copie et installe l’agent sur l’ordinateur distant.

```powershell
$DLoc = "\\REMOTESERVER01\c$\Windows\Temp"
$Agent = "\\REMOTESERVER01\c$\Windows\Temp\MARSAgentInstaller.exe"
$Args = "/q"
Copy-Item "C:\Downloads\MARSAgentInstaller.exe" -Destination $DLoc -Force

$Session = New-PSSession -ComputerName REMOTESERVER01
Invoke-Command -Session $Session -Script { param($D, $A) Start-Process -FilePath $D $A -Wait } -ArgumentList $Agent, $Args
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Sauvegarde Azure pour client/serveur Windows :

* [Présentation d’Azure Backup](./backup-overview.md)
* [Sauvegarder des serveurs Windows](backup-windows-with-mars-agent.md)
