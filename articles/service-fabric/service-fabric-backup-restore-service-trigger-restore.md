---
title: Restauration de la sauvegarde dans Azure Service Fabric
description: Utilisez la fonctionnalité de sauvegarde et de restauration périodiques de Service Fabric pour restaurer des données issues d’une sauvegarde des données de votre application.
author: aagup
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: aagup
ms.openlocfilehash: 3d881033b8dde6cc55a9720ec94084bd876116f1
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207391"
---
# <a name="restoring-backup-in-azure-service-fabric"></a>Restauration de la sauvegarde dans Azure Service Fabric

Dans Azure Service Fabric, les services fiables avec état et Reliable Actors peuvent conserver un état mutable faisant autorité après une transaction demande/réponse. Un service avec état peut tomber en panne pendant une longue période ou perdre des informations en raison d’un incident. Si cela se produit, le service doit être restauré à partir de la dernière sauvegarde acceptable pour continuer de fonctionner.

Par exemple, vous pouvez configurer un service pour qu’il sauvegarde ses données afin de vous protéger des scénarios suivants :

- **Cas de reprise d’activité après sinistre**  : Perte définitive d’un cluster Service Fabric entier.
- **Cas de perte de données**  : Perte définitive de la majorité des réplicas d’une partition de service.
- **Cas de perte de données**  : Suppression accidentelle ou altération du service. Par exemple, un administrateur supprime le service par erreur.
- **Cas d’altération des données**  : Bogues dans le service qui provoquent l’altération des données. Par exemple, l’altération des données peut se produire lorsqu’une mise à niveau de code de service écrit des données erronées dans une collection fiable. Dans ce cas, le code et les données devront peut-être être restaurés à un état antérieur.

## <a name="prerequisites"></a>Prérequis

- Pour déclencher une restauration, le _service FAS (Fault Analysis Service)_ doit être activé pour le cluster.
- Le _service BRS (Backup Restore Service)_ a créé la sauvegarde.
- La restauration peut être déclenchée sur une partition uniquement.
- Installez le module Microsoft.ServiceFabric.Powershell.Http [en préversion] pour effectuer des appels de configuration.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

- Assurez-vous que le cluster est connecté à l’aide de la commande `Connect-SFCluster` avant d’effectuer toute requête de configuration à l’aide du module Microsoft.ServiceFabric.Powershell.Http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```


## <a name="triggered-restore"></a>Restauration déclenchée

Une restauration peut être déclenchée pour tous les scénarios suivants :

- Restauration de données dans le cadre d’une _reprise d’activité_
- Restauration des données après une _altération ou une perte des données_

### <a name="data-restore-in-the-case-of-disaster-recovery"></a>Restauration des données en cas de reprise d’activité après sinistre

Si un cluster Service Fabric est perdu dans son intégralité, vous pouvez récupérer les données des partitions du service fiable avec état et des Reliable Actors. Vous pouvez sélectionner la sauvegarde souhaitée dans la liste lorsque vous utilisez [GetBackupAPI avec les détails de stockage de sauvegarde](/rest/api/servicefabric/sfclient-api-getbackupsfrombackuplocation). L’énumération de sauvegardes peut fonctionner pour une application, un service ou une partition.

Pour l’exemple suivant, nous allons supposer que le cluster perdu soit le même que celui mentionné dans [Activation de la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors)](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors). Dans ce cas, `SampleApp` est déployé avec la stratégie de sauvegarde activée, et les sauvegardes sont configurées sur le stockage Azure.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell
Get-SFBackupsFromBackupLocation -Application -ApplicationName 'fabric:/SampleApp' -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Exécutez un script PowerShell pour utiliser l’API REST afin de retourner la liste des sauvegardes créées pour toutes les partitions de l’application `SampleApp`. L’API a besoin des informations de stockage de sauvegarde afin de lister les sauvegardes disponibles.

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$BackupEntity = @{
    EntityKind = 'Application'
    ApplicationName='fabric:/SampleApp'
}

$BackupLocationAndEntityInfo = @{
    Storage = $StorageInfo
    BackupEntity = $BackupEntity
}

$body = (ConvertTo-Json $BackupLocationAndEntityInfo)
$url = "https://myalternatesfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

Exemple de sortie pour l’exécution ci-dessus :

```
BackupId                : b9577400-1131-4f88-b309-2bb1e943322c
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 20.55.16.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3334
CreationTimeUtc         : 2018-04-06T20:55:16Z
FailureError            : 
*
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
*
BackupId                : 69436834-c810-4163-9386-a7a800f78359
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.25.36.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3764
CreationTimeUtc         : 2018-04-06T21:25:36Z
FailureError            :
```

Pour déclencher la restauration, choisissez l’une des sauvegardes. Par exemple, la sauvegarde en cours pour la reprise d’activité peut être la sauvegarde suivante :

```
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
```

Pour l’API de restauration, vous devez fournir les détails sur _BackupId_ et _BackupLocation_ .

Vous devez également choisir une partition de destination dans le cluster de remplacement, comme indiqué dans le [schéma de partition](service-fabric-concepts-partitioning.md#get-started-with-partitioning). La sauvegarde du cluster de remplacement est restaurée sur la partition spécifiée dans le schéma de partition du cluster d’origine perdu.

Si l’ID de partition du cluster de remplacement est `1c42c47f-439e-4e09-98b9-88b8f60800c6`, vous pouvez le mapper sur l’ID de partition du cluster d’origine `974bd92a-b395-4631-8a7f-53bd4ae9cf22` en comparant la clé haute et la clé basse du _partitionnement par plages de valeurs (UniformInt64Partition)_ .

Pour le _partitionnement nommé_ , la valeur de nom est comparée pour identifier la partition cible dans l’autre cluster.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell

Restore-SFPartition  -PartitionId '1c42c47f-439e-4e09-98b9-88b8f60800c6' -BackupId 'b0035075-b327-41a5-a58f-3ea94b68faa4' -BackupLocation 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip' -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container'

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Pour demander une restauration avec la partition du cluster de sauvegarde, utilisez l’[API Restore](/rest/api/servicefabric/sfclient-api-restorepartition) suivante :

```powershell

$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$RestorePartitionReference = @{
    BackupId = 'b0035075-b327-41a5-a58f-3ea94b68faa4'
    BackupLocation = 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'
    BackupStorage  = $StorageInfo
}

$body = (ConvertTo-Json $RestorePartitionReference) 
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/1c42c47f-439e-4e09-98b9-88b8f60800c6/$/Restore?api-version=6.4" 

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Vous pouvez suivre la progression d’une restauration avec TrackRestoreProgress.

> [!NOTE]
> Lors de l’utilisation de PowerShell pour restaurer une partition, si BackupLocation a la valeur « $ », la séquence d’échappement utilise « ~ »
>

### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer
Il est possible de déclencher une restauration à partir de Service Fabric Explorer. Vérifiez que le mode avancé est activé dans les paramètres Service Fabric Explorer.
1. Sélectionnez les partitions souhaitées, puis cliquez sur Actions. 
2. Sélectionnez Déclencher la restauration de partition, puis renseignez les informations pour Azure :

    ![Déclencher la restauration de partition][2]

    ou FileShare :

    ![Déclencher la restauration de partition pour FileShare][3]

### <a name="data-restore-for-_data-corruption__data-loss_"></a>Restauration des données après une _altération_/_perte de données_

Dans le cas d’une _perte de données_ ou d’une _altération des données_ , vous pouvez restaurer les données des partitions du service fiable avec état et Reliable Actors à l’aide de l’une des sauvegardes choisies.

L’exemple suivant est la continuation du scénario mentionné dans [Activation de la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors)](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors). Dans cet exemple, une stratégie de sauvegarde est activée pour la partition, et le service effectue des sauvegardes à la fréquence souhaitée dans le stockage Azure.

Sélectionnez une sauvegarde à partir de la sortie de [GetBackupAPI](service-fabric-backuprestoreservice-quickstart-azurecluster.md#list-backups). Dans ce scénario, la sauvegarde est générée à partir du même cluster qu’avant.

Pour déclencher la restauration, choisissez une sauvegarde dans la liste. Pour la _perte_/_altération de données_ actuelle, sélectionnez la sauvegarde suivante :

```
BackupId                : b0035075-b327-41a5-a58f-3ea94b68faa4
BackupChainId           : b9577400-1131-4f88-b309-2bb1e943322c
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=974bd92a-b395-4631-8a7f-53bd4ae9cf22}
BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 3552
CreationTimeUtc         : 2018-04-06T21:10:27Z
FailureError            :
```

Pour l’API de restauration, fournissez les détails sur _BackupId_ et _BackupLocation_ . La sauvegarde est activée dans le cluster. Par conséquent, le _service BRS (Backup Restore Service)_ de Service Fabric identifie le bon emplacement de stockage à l’aide de la stratégie de sauvegarde associée.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell
Restore-SFPartition  -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22' -BackupId 'b0035075-b327-41a5-a58f-3ea94b68faa4' -BackupLocation 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

```powershell
$RestorePartitionReference = @{
    BackupId = 'b0035075-b327-41a5-a58f-3ea94b68faa4',
    BackupLocation = 'SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-04-06 21.10.27.zip'
}

$body = (ConvertTo-Json $RestorePartitionReference)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Restore?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

Vous pouvez suivre la progression d’une restauration avec TrackRestoreProgress.

> [!NOTE]
> Lors de l’utilisation de PowerShell pour restaurer une partition, si BackupLocation a la valeur « $ », la séquence d’échappement utilise « ~ »
>

## <a name="track-restore-progress"></a>Suivre la progression de la restauration

Une partition d’un service fiable avec état ou Reliable Actor accepte une seule demande de restauration à la fois. Une partition ne peut accepter une autre demande de restauration qu’une fois la demande en cours terminée. Plusieurs demandes de restauration peuvent être déclenchées en même temps sur des partitions différentes.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell
    Get-SFPartitionRestoreProgress -PartitionId '974bd92a-b395-4631-8a7f-53bd4ae9cf22'
```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetRestoreProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

$restoreResponse = (ConvertFrom-Json $response.Content)
$restoreResponse | Format-List
```

La demande de restauration progresse dans l’ordre suivant :

1. **Accepted** (Acceptée) : l’état de restauration _Accepted_ indique que la demande a été déclenchée avec les bons paramètres de demande.
    ```
    RestoreState  : Accepted
    TimeStampUtc  : 0001-01-01T00:00:00Z
    RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
    RestoredLsn   : 3552
    ```
2. **InProgress** (En cours) : l’état de restauration _InProgress_ indique que la partition fait l’objet d’une restauration avec la sauvegarde mentionnée dans la demande. La partition indique l’état _dataloss_ .
    ```
    RestoreState  : RestoreInProgress
    TimeStampUtc  : 0001-01-01T00:00:00Z
    RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
    RestoredLsn   : 3552
    ```
    
3. **Success** (Réussite), **Failure** (Échec) ou **Timeout** (Expiration du délai) : une restauration demandée peut être effectuée avec l’un des états suivants. Chaque état présente les détails d’importance et de réponse suivants :
    - **Réussite**  : l’état de restauration _Success_ indique un état de partition retrouvée. La partition indique les états _RestoredEpoch_ et _RestoredLSN_ ainsi que l’heure UTC.

        ```
        RestoreState  : Success
        TimeStampUtc  : 2018-11-22T11:22:33Z
        RestoredEpoch : @{DataLossNumber=131675205859825409; ConfigurationNumber=8589934592}
        RestoredLsn   : 3552
        ```        
    - **Échec**  : l’état de restauration _Failure_ indique l’échec de la demande de restauration. La cause de l’échec est signalée.

        ```
        RestoreState  : Failure
        TimeStampUtc  : 0001-01-01T00:00:00Z
        RestoredEpoch : 
        RestoredLsn   : 0
        ```
    - **Timeout** (Expiration du délai) : l’état de restauration _Timeout_ indique que la demande a expiré. Créez une demande de restauration avec une valeur [RestoreTimeout](/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout) supérieure. Le délai d’expiration par défaut est de 10 minutes. Avant de demander une nouvelle restauration, vérifiez que la partition n’est pas dans un état de perte de données (dataloss).
     
        ```
        RestoreState  : Timeout
        TimeStampUtc  : 0001-01-01T00:00:00Z
        RestoredEpoch : 
        RestoredLsn   : 0
        ```

## <a name="automatic-restore"></a>Restauration automatique

Vous pouvez configurer les partitions du service fiable avec état et Reliable Actors dans le cluster Service Fabric pour la _restauration automatique_ . Dans la stratégie de sauvegarde, définissez `AutoRestore` sur _true_ . L’activation de la _restauration automatique_ restaure automatiquement les données de la dernière sauvegarde de partition, si une perte de données est signalée. Pour plus d'informations, consultez les pages suivantes :

- [Activation de la restauration automatique dans la stratégie de sauvegarde](service-fabric-backuprestoreservice-configure-periodic-backup.md#auto-restore-on-data-loss)
- [Référence sur l’API RestorePartition](/rest/api/servicefabric/sfclient-api-restorepartition)
- [Référence sur l’API GetPartitionRestoreProgress](/rest/api/servicefabric/sfclient-api-getpartitionrestoreprogress)

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de la configuration de la sauvegarde périodique](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Informations de référence sur l’API REST de sauvegarde et restauration](/rest/api/servicefabric/sfclient-index-backuprestore)

[2]: ./media/service-fabric-backuprestoreservice/restore-partition-backup.png
[3]: ./media/service-fabric-backuprestoreservice/restore-partition-fileshare.png
