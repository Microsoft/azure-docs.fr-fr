---
title: Sauvegarde/restauration périodiques dans Azure Service Fabric autonome
description: Utilisez la fonctionnalité de sauvegarde et de restauration périodiques d’une instance Service Fabric autonome pour activer la sauvegarde périodique des données de votre application.
ms.topic: conceptual
ms.date: 5/24/2019
ms.openlocfilehash: d20882ba5f7f31ef453c5d28f8bc37155cc99abd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91538583"
---
# <a name="periodic-backup-and-restore-in-a-standalone-service-fabric"></a>Sauvegarde et restauration périodiques dans un Service Fabric autonome
> [!div class="op_single_selector"]
> * [Clusters sur Azure](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [Clusters autonomes](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service Fabric est une plateforme de systèmes distribués qui facilite le développement et la gestion d'applications cloud basées sur des microservices distribués et fiables. Il permet l’exécution de microservices avec et sans état. Les services avec état peuvent conserver un état mutable faisant autorité au-delà de la demande et la réponse ou d’une transaction complète. Si un service avec état tombe en panne pendant une longue période ou perd des informations suite à un sinistre, il peut être nécessaire de le restaurer sur l'état qui était le sien lors d'une sauvegarde récente afin de continuer à assurer le service une fois qu'il sera redevenu opérationnel.

Service Fabric réplique l’état sur plusieurs nœuds afin de garantir une haute disponibilité du service. Même si un nœud du cluster échoue, le service continue d’être disponible. Toutefois, dans certains cas, il est toujours souhaitable que les données de service soient fiables par rapport à des défaillances plus importantes.
 
Par exemple, la sauvegarde des données d'un service peut être nécessaire afin de les protéger des situations suivantes :
- Perte définitive d’un cluster Service Fabric entier.
- Perte définitive de la majorité des réplicas d’une partition de service.
- Erreurs d’administration dans le cadre desquelles l’état est accidentellement supprimé ou endommagé. Par exemple, un administrateur disposant de privilèges suffisants supprime le service par erreur.
- Bogues dans le service qui provoquent l’altération des données. Par exemple, cela peut se produire lorsqu’une mise à niveau de code de service écrit des données erronées dans une collection fiable. Dans ce cas, le code et les données devront peut-être être restaurés à un état antérieur.
- Traitement des données hors connexion. Il peut être utile de disposer du traitement des données hors connexion pour le décisionnel qui a lieu séparément du service qui génère les données.

Service Fabric fournit une API intégrée pour effectuer des opérations de [sauvegarde et restauration](service-fabric-reliable-services-backup-restore.md) dans le temps. Les développeurs d’applications peuvent utiliser ces API pour sauvegarder régulièrement l’état du service. De plus, si les administrateurs de service souhaitent déclencher une sauvegarde depuis l'extérieur du service à un moment donné (par exemple, avant la mise à niveau de l'application), les développeurs doivent exposer la sauvegarde (et la restauration) en tant qu'API du service. La maintenance des sauvegardes constitue un coût supplémentaire. Par exemple, vous souhaitez effectuer cinq sauvegardes incrémentielles toutes les demi-heures, suivies d’une sauvegarde complète. Après la sauvegarde complète, vous pouvez supprimer les sauvegardes incrémentielles précédentes. Cette approche nécessite du code supplémentaire, ce qui entraîne un coût supplémentaire durant le développement d’applications.

La sauvegarde régulière des données d’application est nécessaire à la gestion d’une application distribuée et la protection contre la perte de données ou une perte prolongée de la disponibilité du service. Service Fabric fournit un service de sauvegarde et restauration facultatif, ce qui vous permet de configurer une sauvegarde périodique des services fiables (Reliable Services) avec état (dont les services d’acteur) sans avoir à écrire du code supplémentaire. Il facilite également la restauration des sauvegardes précédemment effectuées. 

Service Fabric fournit un ensemble d’API pour obtenir les fonctions suivantes associées à la fonctionnalité de sauvegarde et restauration périodiques :

- Planifier la sauvegarde périodique des services avec état fiables et des acteurs fiables (Reliable Actors) avec prise en charge pour charger la sauvegarde sur des emplacements de stockage (externes). Emplacements de stockage pris en charge
    - Stockage Azure
    - Partage de fichiers (localement)
- Énumérer les sauvegardes
- Déclencher une sauvegarde non planifiée d'une partition
- Restaurer une partition à l’aide d’une sauvegarde précédente
- Suspendre temporairement les sauvegardes
- Gérer la rétention des sauvegardes (à venir)

## <a name="prerequisites"></a>Prérequis
* Cluster Service Fabric avec Fabric 6.4 ou version ultérieure. Consultez cet [article](service-fabric-cluster-creation-for-windows-server.md) pour la procédure de téléchargement du package nécessaire.
* Certificat X.509 pour le chiffrement des secrets nécessaire pour se connecter au stockage pour stocker les sauvegardes. Consultez [l’article](service-fabric-windows-cluster-x509-security.md) pour savoir comment acquérir ou créer un certificat X.509 auto-signé.

* Application avec état fiable Service Fabric générée avec le kit SDK Service Fabric version 3.0 ou ultérieure. Pour les applications qui ciblent .NET Core 2.0, l'application doit être générée à l'aide du SDK Service Fabric 3.1 ou version ultérieure.
* Installez le module Microsoft.ServiceFabric.PowerShell.Http [en préversion] pour effectuer des appels de configuration.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.PowerShell.Http -AllowPrerelease
```

* Assurez-vous que le cluster est connecté à l'aide de la commande `Connect-SFCluster` avant toute requête de configuration via le module Microsoft.ServiceFabric.PowerShell.Http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>Activation du service de sauvegarde et de restauration
Vous devez d’abord activer le _service de sauvegarde et de restauration_ dans votre cluster. Récupérez le modèle approprié pour le cluster que vous souhaitez déployer. Vous pouvez utiliser les [exemples de modèles](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples). Activez le _service de sauvegarde et de restauration_ en effectuant les étapes suivantes :

1. Vérifiez si `apiversion` a la valeur `10-2017` dans le fichier de configuration de cluster et, si ce n’est pas le cas, procédez à une mise à jour, comme indiqué dans l’extrait de code suivant :

    ```json
    {
        "apiVersion": "10-2017",
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        ...
    }
    ```

2. À présent, activez le _service de sauvegarde et de restauration_ en ajoutant la section `addonFeatures` suivante sous la section `properties`, comme indiqué dans l’extrait de code ci-après : 

    ```json
        "properties": {
            ...
            "addonFeatures": ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```

3. Configurez un certificat X.509 pour le chiffrement des informations d’identification. Ceci est important afin de vous assurer que les informations d’identification fournies, le cas échéant, pour vous connecter au stockage sont chiffrées avant d’être rendues persistantes. Configurez le certificat de chiffrement en ajoutant la section `BackupRestoreService` suivante sous la section `fabricSettings`, comme indiqué dans l’extrait de code ci-après : 

    ```json
    "properties": {
        ...
        "addonFeatures": ["BackupRestoreService"],
        "fabricSettings": [{
            "name": "BackupRestoreService",
            "parameters":  [{
                "name": "SecretEncryptionCertThumbprint",
                "value": "[Thumbprint]"
            }]
        }
        ...
    }
    ```

4. Après avoir mis à jour votre fichier de configuration de cluster avec les modifications précédentes, appliquez-les et laissez le déploiement/la mise à niveau s'accomplir. Une fois l’opération terminée, le _service de sauvegarde et de restauration_ commence à s’exécuter dans votre cluster. L’URI de ce service est `fabric:/System/BackupRestoreService` et le service peut être situé sous la section du service système dans Service Fabric Explorer. 



## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>Activation de la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors)
Examinons la procédure pour activer la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors). Cette procédure suppose les éléments suivants :
- Le cluster est configuré avec le service de sauvegarde et restauration.
- Un service avec état fiable est déployé sur le cluster. Pour les besoins de ce guide de démarrage rapide, l’URI de l’application est `fabric:/SampleApp` et l’URI du service avec état fiable appartenant à cette application est `fabric:/SampleApp/MyStatefulService`. Ce service est déployé avec une partition unique et l’ID de partition est `23aebc1e-e9ea-4e16-9d5c-e91a614fefa7`.  

### <a name="create-backup-policy"></a>Créer la stratégie de sauvegarde

La première étape consiste à créer la stratégie de sauvegarde qui décrit la planification de la sauvegarde, le stockage cible pour les données de sauvegarde, le nom de la stratégie, les sauvegardes incrémentielles maximales autorisées avant le déclenchement de la sauvegarde complète et la stratégie de conservation pour le stockage des sauvegardes. 

Pour le stockage de sauvegarde, créez le partage de fichiers et accordez un accès Lecture/écriture à ce partage pour tous les ordinateurs du nœud Service Fabric. Cet exemple suppose que le partage nommé `BackupStore` est présent sur `StorageServer`.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.PowerShell.Http

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -FileShare -Path '\\StorageServer\BackupStore' -Basic -RetentionDuration '10.00:00:00'

```
#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Exécutez le script PowerShell suivant pour appeler l’API REST requise afin de créer une stratégie.

```powershell
$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
}   

$StorageInfo = @{
    Path = '\\StorageServer\BackupStore'
    StorageKind = 'FileShare'
}

$RetentionPolicy = @{ 
    RetentionPolicyType = 'Basic'
    RetentionDuration =  'P10D'
}

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
    RetentionPolicy = $RetentionPolicy
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "http://localhost:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
```

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

1. Dans Service Fabric Explorer, accédez à l’onglet Sauvegardes et sélectionnez Actions > Créer une stratégie de sauvegarde.

    ![Créer la stratégie de sauvegarde][6]

2. Renseignez les informations. Pour les clusters autonomes, FileShare doit être sélectionné.

    ![Créer une stratégie de sauvegarde FileShare][7]

### <a name="enable-periodic-backup"></a>Activer la sauvegarde périodique
Après avoir défini la stratégie de sauvegarde pour répondre aux exigences de protection des données de l’application, la stratégie doit être associée à l’application. Selon les besoins, la stratégie de sauvegarde peut être associée à une application, un service ou une partition.


#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.PowerShell.Http

```powershell
Enable-SFApplicationBackup -ApplicationId 'SampleApp' -BackupPolicyName 'BackupPolicy1'
```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell
Exécutez le script PowerShell suivant pour appeler l’API REST requise afin d’associer la stratégie de sauvegarde nommée `BackupPolicy1` créée à l’étape ci-dessus à l’application `SampleApp`.

```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "http://localhost:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
``` 

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

1. Sélectionnez une application et accédez à Action. Cliquez sur Activer/Mettre à jour la sauvegarde de l’application.

    ![Activer la sauvegarde de l’application][3] 

2. Enfin, sélectionnez la stratégie souhaitée, puis choisissez *Activer la sauvegarde*.

    ![Sélectionner une stratégie][4]

### <a name="verify-that-periodic-backups-are-working"></a>Vérifier le fonctionnement des sauvegardes périodiques

Après avoir activé la sauvegarde pour l’application, toutes les partitions appartenant à des services avec état fiables et des acteurs fiables (Reliable Actors) sous l’application commencent à être sauvegardées régulièrement conformément à la stratégie de sauvegarde associée.

![Événement d’intégrité de partition sauvegardée][0]

### <a name="list-backups"></a>Répertorier les sauvegardes

Les sauvegardes associées à toutes les partitions appartenant à des services avec état fiables et des acteurs fiables (Reliable Actors) de l’application peuvent être énumérées à l’aide de l’API _GetBackups_. Selon les besoins, les sauvegardes peuvent être énumérées pour une application, un service ou une partition.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.PowerShell.Http

```powershell
    Get-SFApplicationBackupList -ApplicationId WordCount     
```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Exécutez le script PowerShell suivant pour appeler l’API HTTP afin d’énumérer les sauvegardes créées pour toutes les partitions à l’intérieur de l’application `SampleApp`.

```powershell
$url = "http://localhost:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get

$BackupPoints = (ConvertFrom-Json $response.Content)
$BackupPoints.Items
```

Exemple de sortie pour l’exécution ci-dessus :

```
BackupId                : d7e4038e-2c46-47c6-9549-10698766e714
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.39.40.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2058
CreationTimeUtc         : 2018-04-01T19:39:40Z
FailureError            : 

BackupId                : 8c21398a-2141-4133-b4d7-e1a35f0d7aac
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.54.38.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2237
CreationTimeUtc         : 2018-04-01T19:54:38Z
FailureError            : 

BackupId                : fc75bd4c-798c-4c9a-beee-e725321f73b2
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 20.09.44.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2437
CreationTimeUtc         : 2018-04-01T20:09:44Z
FailureError            : 
```

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

Pour afficher les sauvegardes dans Service Fabric Explorer, accédez à une partition et sélectionnez l’onglet Sauvegardes.

![Énumérer les sauvegardes][5]

## <a name="limitation-caveats"></a>Limitations/mises en garde
- Les applets de commande PowerShell Service Fabric sont en mode Aperçu.
- Pas de prise en charge des clusters Service Fabric sur Linux.

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de la configuration de la sauvegarde périodique](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Informations de référence sur l’API REST de sauvegarde et restauration](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/partition-backedup-health-event.png
[3]: ./media/service-fabric-backuprestoreservice/enable-app-backup.png
[4]: ./media/service-fabric-backuprestoreservice/enable-application-backup.png
[5]: ./media/service-fabric-backuprestoreservice/backup-enumeration.png
[6]: ./media/service-fabric-backuprestoreservice/create-bp.png
[7]: ./media/service-fabric-backuprestoreservice/create-bp-fileshare.png
