---
title: Sauvegarde à la demande dans Azure Service Fabric
description: Utilisez la fonctionnalité de sauvegarde et de restauration de Service Fabric pour sauvegarder vos données d’application selon vos besoins.
author: aagup
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: aagup
ms.openlocfilehash: 04d8bb4a9f8157a229751d073e8d351f5448fa68
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86247895"
---
# <a name="on-demand-backup-in-azure-service-fabric"></a>Sauvegarde à la demande dans Azure Service Fabric

Vous pouvez sauvegarder les données des services fiables avec état et Reliable Actors pour pallier les sinistres et la perte de données.

Azure Service Fabric comprend des fonctionnalités pour la [sauvegarde périodique des données](service-fabric-backuprestoreservice-quickstart-azurecluster.md) et la sauvegarde des données selon les besoins. La sauvegarde à la demande est utile car elle protège de la _perte des données_/_l’altération des données_ causées par des changements prévus dans le service sous-jacent ou son environnement.

Les fonctionnalités de sauvegarde à la demande sont utiles pour capturer l’état des services avant que vous ne déclenchiez manuellement un service ou une opération de l’environnement du service. Par exemple, si vous apportez une modification dans les fichiers binaires du service lors du passage à une version antérieure ou ultérieure du service. Dans ce cas, la sauvegarde à la demande peut protéger les données d’une éventuelle altération due aux bogues du code de l’application.
## <a name="prerequisites"></a>Prérequis

- Installez le module Microsoft.ServiceFabric.Powershell.Http [en préversion] pour effectuer des appels de configuration.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

- Assurez-vous que le cluster est connecté à l’aide de la commande `Connect-SFCluster` avant d’effectuer toute requête de configuration à l’aide du module Microsoft.ServiceFabric.Powershell.Http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```


## <a name="triggering-on-demand-backup"></a>Déclenchement d’une sauvegarde à la demande

La sauvegarde à la demande nécessite des informations de stockage pour charger les fichiers de sauvegarde. Vous devez spécifier l’emplacement de sauvegarde à la demande, soit dans la stratégie de sauvegarde périodique, soit dans une requête de sauvegarde à la demande.

### <a name="on-demand-backup-to-storage-specified-by-a-periodic-backup-policy"></a>Sauvegarde à la demande dans le stockage spécifié dans une stratégie de sauvegarde périodique

Vous pouvez configurer la stratégie de sauvegarde périodique de manière à utiliser une partition d’un service fiable avec état ou Reliable Actor, afin d’effectuer une sauvegarde à la demande supplémentaire vers le stockage.

Le cas suivant est la continuation du scénario mentionné dans [Activation de la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors)](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors). Dans ce cas, vous permettez à une stratégie de sauvegarde d’utiliser une partition et à une sauvegarde d’avoir lieu à une fréquence définie dans le stockage Azure.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' 

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Utilisez l’API [BackupPartition](/rest/api/servicefabric/sfclient-api-backuppartition) pour configurer le déclenchement de la sauvegarde à la demande pour l’ID de partition `974bd92a-b395-4631-8a7f-53bd4ae9cf22`.

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Utilisez l’API [GetBackupProgress](/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) pour activer le suivi de la [progression de la sauvegarde à la demande](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress).

### <a name="on-demand-backup-to-specified-storage"></a>Sauvegarde à la demande dans le stockage spécifié

Vous pouvez demander une sauvegarde à la demande pour une partition d’un service fiable avec état ou d’un Reliable Actor. Fournissez les informations de stockage dans le cadre de la demande de sauvegarde à la demande.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell

Backup-SFPartition -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' -AzureBlobStore -ConnectionString  'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Utilisez l’API [BackupPartition](/rest/api/servicefabric/sfclient-api-backuppartition) pour configurer le déclenchement de la sauvegarde à la demande pour l’ID de partition `974bd92a-b395-4631-8a7f-53bd4ae9cf22`. Incluez les informations de stockage Azure suivantes :

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$OnDemandBackupRequest = @{
    BackupStorage = $StorageInfo
}

$body = (ConvertTo-Json $OnDemandBackupRequest)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Vous pouvez utiliser l’API [GetBackupProgress](/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) pour configurer le suivi de la [progression de la sauvegarde à la demande](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress).

### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer
Vérifiez que le mode avancé est activé dans les paramètres Service Fabric Explorer.
1. Sélectionnez les partitions souhaitées, puis cliquez sur Actions. 
2. Sélectionnez Déclencher la sauvegarde de partition, puis renseignez les informations pour Azure :

    ![Déclencher la sauvegarde de partition][0]

    ou FileShare :

    ![Déclencher la sauvegarde de partition pour FileShare][1]

## <a name="tracking-on-demand-backup-progress"></a>Suivi de la progression de la sauvegarde à la demande

Une partition d’un service fiable avec état ou Reliable Actor accepte une seule demande de sauvegarde à la demande à la fois. Une autre demande ne peut être acceptée que lorsque la demande de sauvegarde à la demande en cours est terminée.

Plusieurs partitions peuvent déclencher simultanément des demandes de sauvegarde à la demande.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell

Get-SFPartitionBackupProgress -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22'

```
#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetBackupProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3' 
$backupResponse = (ConvertFrom-Json $response.Content) 
$backupResponse
```

Les demandes de sauvegarde à la demande peuvent avoir les états suivants :

- **Accepted** (Acceptée) : la sauvegarde a démarré et est en cours sur la partition.
  ```
  BackupState             : Accepted
  TimeStampUtc            : 0001-01-01T00:00:00Z
  BackupId                : 00000000-0000-0000-0000-000000000000
  BackupLocation          :
  EpochOfLastBackupRecord :
  LsnOfLastBackupRecord   : 0
  FailureError            :
  ```
- **Success** (Réussite), **Failure** (Échec) ou **Timeout** (Expiration du délai) : une demande de sauvegarde à la demande peut être effectuée avec l’un des états suivants :
  - **Réussite** : L’état de sauvegarde _Success_ indique que la partition a été sauvegardée avec succès. La réponse fournit _BackupEpoch_ et _BackupLSN_ pour la partition, ainsi que l’heure UTC.
    ```
    BackupState             : Success
    TimeStampUtc            : 2018-11-21T20:00:01Z
    BackupId                : 5d64b697-6acd-45a4-adbd-3d75e0078081
    BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-11-21 20.00.01.zip
    EpochOfLastBackupRecord : @{DataLossNumber=131873018908156893; ConfigurationNumber=8589934592}
    LsnOfLastBackupRecord   : 36
    FailureError            :
    ```
  - **Échec** : L’état de sauvegarde _Failure_ indique qu’un échec s’est produit lors de la sauvegarde de l’état de la partition. La cause de l’échec est indiquée dans la réponse.
    ```
    BackupState             : Failure
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_BACKUPCOPIER_UNEXPECTED_ERROR; Message=An error occurred during this operation.  Please check the trace logs for more details.}
    ```
  - **Timeout** (Expiration du délai) : l’état de sauvegarde_Timeout_ indique que la sauvegarde d’état de partition n’a pas pu être créée dans un laps de temps donné. Le délai d’expiration par défaut est de 10 minutes. Dans ce scénario, effectuez une nouvelle demande de sauvegarde à la demande avec une valeur de [BackupTimeout](/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout) supérieure.
    ```
    BackupState             : Timeout
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_TIMEOUT; Message=The request of backup has timed out.}
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre la configuration de la sauvegarde périodique](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Référence sur l’API REST BackupRestore](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/trigger-partition-backup.png
[1]: ./media/service-fabric-backuprestoreservice/trigger-backup-fileshare.png
