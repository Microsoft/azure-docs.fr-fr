---
title: Utiliser des scripts AzureRM PowerShell pour gérer les appareils StorSimple
description: Apprenez à utiliser les scripts basés sur le kit de développement logiciel (SDK) Azure Resource Manager pour gérer votre appareil de la gamme StorSimple 8000.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/03/2017
ms.author: alkohli
ms.openlocfilehash: e3339f883d177366a64187e0c090752527802c8d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968772"
---
# <a name="use-azure-resource-manager-sdk-based-scripts-to-manage-storsimple-devices"></a>Utiliser des scripts basés sur le kit de développement logiciel (SDK) Azure Resource Manager pour gérer les appareils StorSimple

Cet article décrit comment les scripts basés sur le kit de développement logiciel (SDK) Azure Resource Manager peuvent être utilisés pour gérer votre appareil de la gamme StorSimple 8000. Un exemple de script est également inclus pour vous guider tout au long des étapes de configuration de votre environnement pour exécuter ces scripts.

Cet article s’applique aux appareils de la gamme StorSimple 8000 s’exécutant dans le portail Azure uniquement.

## <a name="sample-scripts"></a>Exemples de scripts

Les exemples de scripts suivants sont disponibles pour automatiser divers travaux StorSimple.

#### <a name="table-of-azure-resource-manager-sdk-based-sample-scripts"></a>Tableaux des exemples de scripts basés sur le kit de développement logiciel (SDK) Azure Resource Manager

| Script Azure Resource Manager                    | Description                                                                                                                                                                                                       |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Authorize-ServiceEncryptionRollover.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Authorize-ServiceEncryptionRollover.ps1)          | Ce script vous permet d’autoriser votre appareil StorSimple à modifier la clé de chiffrement des données de service.                                                                                                           |
| [Create-StorSimpleCloudAppliance.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Create-StorSimpleCloudAppliance.ps1)              | Ce script crée une StorSimple Cloud Appliance 8010 ou 8020. L’appliance cloud peut ensuite être configurée et inscrite auprès de votre service StorSimple Data Manager.                                                       |
| [CreateOrUpdate-Volume.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/CreateOrUpdate-Volume.ps1)                        | Ce script crée ou modifie des volumes StorSimple.                                                                                                                                                             |
| [Get-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackup.ps1)                             | Ce script répertorie toutes les sauvegardes d’un appareil inscrit auprès de votre service StorSimple Data Manager.                                                                                                          |
| [Get-DeviceBackupPolicy.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackupPolicy.ps1)                       | Ce script répertorie toutes les stratégies de sauvegarde de votre appareil StorSimple.                                                                                                                                                 |
| [Get-DeviceJobs.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1)                               | Ce script obtient tous les travaux StorSimple s’exécutant sur votre service StorSimple Device Manager.                                                                                                                     |
| [Get-DeviceUpdateAvailability.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceUpdateAvailability.ps1)                 | Ce script analyse le serveur Update et vous permet de savoir si des mises à jour sont disponibles pour votre appareil StorSimple.                                                                                          |
| [Install-DeviceUpdate.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Install-DeviceUpdate.ps1)                         | Ce script installe les mises à jour disponibles sur votre appareil StorSimple.                                                                                                                                           |
| [Manage-CloudSnapshots.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Manage-CloudSnapshots.ps1)                        | Ce script démarre une capture instantanée cloud manuelle et supprime les captures instantanées cloud plus anciennes que la durée de rétention en jours spécifiée.                                                                                                   |
| [Monitor-Backups.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1)                              | Ce script PowerShell de Runbook Azure Automation indique l’état de tous les travaux de sauvegarde.                                                                                                              |
| [Remove-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Remove-DeviceBackup.ps1)                          | Ce script supprime un objet de sauvegarde unique.                                                                                                                                                           |
| [Start-DeviceBackupJob.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Start-DeviceBackupJob.ps1)                        | Ce script démarre une sauvegarde manuelle sur votre appareil StorSimple.                                                                                                                                       |
| [Update-CloudApplianceServiceEncryptionKey.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Update-CloudApplianceServiceEncryptionKey.ps1)    | Ce script met à jour la clé de chiffrement des données de service pour toutes les StorSimple Cloud Appliances 8010/8020 inscrites auprès de votre service StorSimple Device Manager.                                     |
| [Verify-BackupScheduleAndBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Verify-BackupScheduleAndBackup.ps1)               | Ce script met en évidence les sauvegardes manquantes après l’analyse de toutes les planifications associées aux stratégies de sauvegarde. Il vérifie également le catalogue de sauvegardes avec la liste des sauvegardes disponibles.             |




## <a name="configure-and-run-a-sample-script"></a>Configurer et exécuter un exemple de script

Cette section utilise un exemple de script et décrit en détail les différentes étapes requises pour exécuter ce script.

### <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, assurez-vous de satisfaire les exigences suivantes :

*   Azure PowerShell installé. Pour installer les modules Azure PowerShell :
    * Dans un environnement Windows, suivez les étapes décrites dans [Install and configure Azure PowerShell](/powershell/azure/install-az-ps) (Installer et configurer Azure PowerShell). Vous pouvez installer Azure PowerShell sur votre hôte Windows Server pour votre StorSimple si vous utilisez un.
    * Dans un environnement Linux ou MacOS, suivez les étapes décrites dans [Install and configure Azure PowerShell on macOS and Linux](/powershell/azure/install-az-ps) (Installer et configurer Azure PowerShell sur MacOS ou Linux).

Pour plus d’informations sur l’utilisation d’Azure PowerShell, consultez [Getting started with Azure PowerShell](/powershell/azure/get-started-azureps) (Prise en main d’Azure PowerShell).

### <a name="run-azure-powershell-script"></a>Exécuter le script Azure PowerShell

Le script utilisé dans cet exemple répertorie tous les travaux sur un appareil StorSimple. Cela inclut les travaux qui ont réussi, qui ont échoué ou qui sont en cours d’exécution. Procédez comme suit pour télécharger et exécuter le script.

1. Lancez Azure PowerShell. Créez un dossier et basculez vers ce nouveau dossier.

    ```
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```    
2. [Télécharger l’interface de ligne de commande NuGet](https://www.nuget.org/downloads) sous le dossier créé à l’étape précédente. Il existe différentes versions de _nuget.exe_. Choisissez la version correspondant à votre kit de développement logiciel (SDK). Chaque lien de téléchargement pointe directement vers un fichier _.exe_. Veillez à cliquer avec le bouton droit et à enregistrer le fichier sur votre ordinateur plutôt que de l’exécuter à partir du navigateur.

    Vous pouvez également exécuter la commande suivante pour télécharger et stocker le script dans le même dossier que vous avez créé précédemment.
    
    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```
3. Téléchargez le kit de développement logiciel (SDK) dépendant.

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```    
4. Téléchargez le script à partir de l’exemple de projet GitHub.

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1

    ```

5. Exécutez le script. Lorsque vous êtes invité à vous authentifier, fournissez vos informations d’identification Azure. Ce script doit générer une liste filtrée de tous les travaux sur votre appareil StorSimple.
           
    ```           
        .\Get-StorSimpleJob.ps1 -SubscriptionId [subid] -TenantId [tenant id] -DeviceName [name of device] -ResourceGroupName [name of resource group] -ManagerName[name of device manager] -FilterByStatus [Filter for job status] -FilterByJobType [Filter for job type] -FilterByStartTime [Filter for start date time] -FilterByEndTime [Filter for end date time]

    ```

### <a name="sample-output"></a>Exemple de sortie

La sortie suivante est présentée lors de l’exécution de l’exemple de script. La sortie contient tous les travaux qui s’exécutaient sur un appareil inscrit, démarré le 25 septembre 2017 et arrêté le 2 octobre 2017.

```
-----------------------------------------
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Scripts\StorSimpleSDKTools> wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1
PS C:\Scripts\StorSimpleSDKTools> .\Get-DeviceJobs.ps1 -SubscriptionId 1234ab5c-678d-910e-9fc4-0accc9c0166e -TenantId 12a345bc-67d8-91ef-01ab-2c7cd123ef45 -DeviceName 8600-ABC1234567D89EF -ResourceGroupName Contoso -ManagerName ContosoDeviceMgr -FilterByStartTime "09/25/2017 08:10:02" -FilterByEndTime "10/02/2017 08:10:02"


Status            : Succeeded
StartTime         : 10/2/2017 10:30:02 PM
EndTime           : 10/2/2017 10:31:36 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : ss-asr-policy1
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC/jobs/75905c48-b153-4af1-8b21-4b9a2ff9
                    825b
Name              : 75905c48-b153-4af1-8b21-4b9a2ff9825b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000
-------------------------------------------
CUT              CUT  
-------------------------------------------
Status            : Succeeded
StartTime         : 9/26/2017 5:00:02 PM
EndTime           : 9/26/2017 5:01:20 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : 8010 policy
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF/jobs/3cfd8108-db60-4e9a-a8da-6d8fe457
                    8d2b
Name              : 3cfd8108-db60-4e9a-a8da-6d8fe4578d2b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000



PS C:\Scripts\StorSimpleSDKTools>
--------------------------------------------

```


## <a name="next-steps"></a>Étapes suivantes

[Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).