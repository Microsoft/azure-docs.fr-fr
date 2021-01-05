---
title: Sauvegarde Azure pour SQL Server s’exécutant sur une machine virtuelle Azure
description: Dans cet article, découvrez comment inscrire Sauvegarde Azure dans SQL Server s’exécutant sur une machine virtuelle Azure.
author: dcurwin
manager: carmonm
ms.service: virtual-machines
ms.subservice: extensions
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: dacurwin
ms.openlocfilehash: 66675f77d480ce8d9f21e5ffb507c475337c9dab
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490728"
---
# <a name="azure-backup-for-sql-server-running-in-azure-vm"></a>Sauvegarde Azure pour SQL Server s’exécutant sur une machine virtuelle Azure

Sauvegarde Azure, entre autres offres, prend en charge la sauvegarde des charges de travail telles que SQL Server s’exécutant sur des machines virtuelles Azure. L’application SQL s’exécutant dans une machine virtuelle Azure, le service de sauvegarde a besoin d’autorisations pour accéder à l’application et récupérer les informations nécessaires.
Pour ce faire, Sauvegarde Azure installe l'extension **AzureBackupWindowsWorkload** sur la machine virtuelle dans laquelle le serveur SQL Server est en cours d’exécution, lors du processus d’inscription déclenché par l’utilisateur.

## <a name="prerequisites"></a>Conditions préalables requises

Pour obtenir la liste des scénarios pris en charge, reportez-vous à la [matrice de prise en charge](../../backup/sql-support-matrix.md#scenario-support) Sauvegarde Azure.

## <a name="network-connectivity"></a>Connectivité réseau

Sauvegarde Azure prend en charge les balises de groupe de sécurité réseau, le déploiement d’un serveur proxy ou les plages d’adresses IP répertoriées. Pour plus d’informations sur chaque méthode, consultez cet [article](../../backup/backup-sql-server-database-azure-vms.md#establish-network-connectivity).

## <a name="extension-schema"></a>Schéma d’extensions

Les schémas d'extensions et les valeurs des propriétés correspondent aux valeurs de configuration (paramètres d’exécution) que le service transmet à l'API CRP. Ces valeurs de configuration sont utilisées lors de l’inscription et de la mise à niveau. L’extension **AzureBackupWindowsWorkload** utilise également ce schéma. Le schéma est prédéfini ; un nouveau paramètre peut être ajouté dans le champ objectStr

  ```json
      "runtimeSettings": [{
      "handlerSettings": {
      "protectedSettingsCertThumbprint": "",
      "protectedSettings": {
      "objectStr": "",
      "logsBlobUri": "",
      "statusBlobUri": ""
      }
      },
      "publicSettings": {
      "locale": "en-us",
      "taskId": "1c0ae461-9d3b-418c-a505-bb31dfe2095d",
      "objectStr": "",
      "commandStartTimeUTCTicks": "636295005824665976",
      "vmType": "vmType"
      }
      }]
      }
  ```

Le JSON suivant illustre le schéma de l’extension WorkloadBackup.  

  ```json
  {
    "type": "extensions",
    "name": "WorkloadBackup",
    "location":"<myLocation>",
    "properties": {
      "publisher": "Microsoft.RecoveryServices",
      "type": "AzureBackupWindowsWorkload",
      "typeHandlerVersion": "1.1",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "locale":"<location>",
        "taskId":"<TaskId used by Azure Backup service to communicate with extension>",

        "objectStr": "<The configuration passed by Azure Backup service to extension>",

        "commandStartTimeUTCTicks": "<Scheduled start time of registration or upgrade task>",
        "vmType": "<Type of VM where registration got triggered Eg. Compute or ClassicCompute>"
      },
      "protectedSettings": {
        "objectStr": "<The sensitive configuration passed by Azure Backup service to extension>",
        "logsBlobUri": "<blob uri where logs of command execution by extension are written to>",
        "statusBlobUri": "<blob uri where status of the command executed by extension is written>"
      }
    }
  }
  ```

### <a name="property-values"></a>Valeurs de propriétés

Name | Valeur/Exemple | Type de données
 --- | --- | ---
locale | fr-FR  |  string
taskId | "1c0ae461-9d3b-418c-a505-bb31dfe2095d"  | string
objectStr <br/> (publicSettings)  | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ==" | string
commandStartTimeUTCTicks | "636967192566036845"  | string
vmType  | "microsoft.compute/virtualmachines"  | string
objectStr <br/> (protectedSettings) | "eyJjb250YWluZXJQcm9wZXJ0aWVzIjp7IkNvbnRhaW5lcklEIjoiMzVjMjQxYTItOGRjNy00ZGE5LWI4NTMtMjdjYTJhNDZlM2ZkIiwiSWRNZ210Q29udGFpbmVySWQiOjM0NTY3ODg5LCJSZXNvdXJjZUlkIjoiMDU5NWIwOGEtYzI4Zi00ZmFlLWE5ODItOTkwOWMyMGVjNjVhIiwiU3Vic2NyaXB0aW9uSWQiOiJkNGEzOTliNy1iYjAyLTQ2MWMtODdmYS1jNTM5ODI3ZTgzNTQiLCJVbmlxdWVDb250YWluZXJOYW1lIjoiODM4MDZjODUtNTQ4OS00NmNhLWEyZTctNWMzNzNhYjg3OTcyIn0sInN0YW1wTGlzdCI6W3siU2VydmljZU5hbWUiOjUsIlNlcnZpY2VTdGFtcFVybCI6Imh0dHA6XC9cL015V0xGYWJTdmMuY29tIn1dfQ==" | string
logsBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Logs.txt?sv=2014-02-14&sr=b&sig=DbwYhwfeAC5YJzISgxoKk%2FEWQq2AO1vS1E0rDW%2FlsBw%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | string
statusBlobUri | <https://seapod01coord1exsapk732.blob.core.windows.net/bcdrextensionlogs-d45d8a1c-281e-4bc8-9d30-3b25176f68ea/sopattna-vmubuntu1404ltsc.v2.Status.txt?sv=2014-02-14&sr=b&sig=96RZBpTKCjmV7QFeXm5IduB%2FILktwGbLwbWg6Ih96Ao%3D&st=2017-11-09T14%3A33%3A29Z&se=2017-11-09T17%3A38%3A29Z&sp=rw> | string

## <a name="template-deployment"></a>Déploiement de modèle

Nous vous recommandons d'ajouter l'extension AzureBackupWindowsWorkload à une machine virtuelle en activant la sauvegarde SQL Server sur la machine virtuelle. Pour ce faire, vous pouvez utiliser le [modèle Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-recovery-services-vm-workload-backup) conçu pour automatiser la sauvegarde sur une machine virtuelle SQL Server.

## <a name="powershell-deployment"></a>Déploiement PowerShell

Pour ce faire, vous devez inscrire la machine virtuelle Azure qui contient l’application SQL avec un coffre Recovery Services. Lors de l’inscription, l'extension AzureBackupWindowsWorkload est installée sur la machine virtuelle. Utilisez la cmdlet  [Register-AzRecoveryServicesBackupContainerPS](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer?view=azps-1.5.0) pour inscrire la machine virtuelle.

```powershell
$myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

La commande renvoie un **conteneur de sauvegarde** de cette ressource et l’état est défini sur **inscrit** .

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus](../../backup/backup-sql-server-azure-troubleshoot.md) sur les instructions de résolution des problèmes liés à la sauvegarde de la machine virtuelle Azure SQL Server
- [Questions courantes](../../backup/faq-backup-sql-server.md) sur la sauvegarde de bases de données SQL Server s’exécutant sur des machines virtuelles Azure et utilisant le service Sauvegarde Azure.
