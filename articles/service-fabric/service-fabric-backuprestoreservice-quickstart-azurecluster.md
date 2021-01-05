---
title: Sauvegarde et restauration périodiques dans Azure Service Fabric
description: Utilisez la fonctionnalité de sauvegarde et de restauration périodiques de Service Fabric pour activer la sauvegarde périodique des données de votre application.
ms.topic: conceptual
ms.date: 5/24/2019
ms.openlocfilehash: 18d10b365cb2e4f4b4e3592233d5f467714bd5b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91538668"
---
# <a name="periodic-backup-and-restore-in-an-azure-service-fabric-cluster"></a>Sauvegarde et restauration périodiques dans un cluster Azure Service Fabric
> [!div class="op_single_selector"]
> * [Clusters sur Azure](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [Clusters autonomes](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service Fabric est une plateforme de systèmes distribués qui facilite le développement et la gestion d’applications cloud basées sur des microservices distribués et fiables. Il permet l’exécution de microservices avec et sans état. Les services avec état peuvent conserver un état mutable faisant autorité au-delà de la demande et la réponse ou d’une transaction complète. Si un service avec état tombe en panne pendant un long moment ou perd des informations en raison d’un sinistre, il doit peut-être être restauré à l’état qu’il avait lors d’une sauvegarde récente afin de continuer à assurer le service une fois redevenu opérationnel.

Service Fabric réplique l’état sur plusieurs nœuds afin de garantir une haute disponibilité du service. Même si un nœud du cluster échoue, le service continue d’être disponible. Toutefois, dans certains cas, il est toujours souhaitable que les données de service soient fiables par rapport à des défaillances plus importantes.
 
Par exemple, le service peut souhaiter sauvegarder ses données afin de les protéger contre les scénarios suivants :
- Perte définitive de la totalité d’un cluster Service Fabric.
- Perte définitive de la majorité des réplicas d’une partition de service.
- Erreurs d’administration dans le cadre desquelles l’état est accidentellement supprimé ou endommagé. Par exemple, un administrateur disposant de privilèges suffisants supprime le service par erreur.
- Bogues dans le service qui provoquent l’altération des données. Par exemple, cela peut se produire lorsqu’une mise à niveau de code de service écrit des données erronées dans une collection fiable. Dans ce cas, le code et les données devront peut-être être restaurés à un état antérieur.
- Traitement des données hors connexion. Il peut être utile de disposer du traitement des données hors connexion pour le décisionnel qui a lieu séparément du service qui génère les données.

Service Fabric fournit une API intégrée pour effectuer des opérations de [sauvegarde et restauration](service-fabric-reliable-services-backup-restore.md) dans le temps. Les développeurs d’applications peuvent utiliser ces API pour sauvegarder régulièrement l’état du service. De plus, si les administrateurs de service souhaitent déclencher une sauvegarde depuis l’extérieur du service à un moment donné, comme avant la mise à niveau de l’application, les développeurs doivent exposer la sauvegarde (et la restauration) en tant qu’API du service. La maintenance des sauvegardes constitue un coût supplémentaire. Par exemple, vous souhaitez effectuer cinq sauvegardes incrémentielles toutes les demi-heures, suivies d’une sauvegarde complète. Après la sauvegarde complète, vous pouvez supprimer les sauvegardes incrémentielles précédentes. Cette approche nécessite du code supplémentaire, ce qui entraîne un coût supplémentaire durant le développement d’applications.

Le service de sauvegarde et de restauration de Service Fabric permet de sauvegarder facilement et automatiquement les informations stockées dans les services avec état. Une sauvegarde périodique des données d’application est essentielle pour éviter la perte de données et l’indisponibilité du service. Service Fabric fournit un service de sauvegarde et restauration facultatif, ce qui vous permet de configurer une sauvegarde périodique des services fiables (Reliable Services) avec état (dont les services d’acteur) sans avoir à écrire du code supplémentaire. Il facilite également la restauration des sauvegardes précédemment effectuées. 


Service Fabric fournit un ensemble d’API pour obtenir les fonctions suivantes associées à la fonctionnalité de sauvegarde et restauration périodiques :

- Planifier la sauvegarde périodique des services avec état fiables et des acteurs fiables (Reliable Actors) avec prise en charge pour charger la sauvegarde sur des emplacements de stockage (externes). Emplacements de stockage pris en charge
    - Stockage Azure
    - Partage de fichiers (localement)
- Énumérer les sauvegardes
- Déclencher une sauvegarde ad hoc d’une partition
- Restaurer une partition à l’aide d’une sauvegarde précédente
- Suspendre temporairement les sauvegardes
- Gérer la rétention des sauvegardes (à venir)

## <a name="prerequisites"></a>Conditions préalables requises
* Cluster Service Fabric avec Fabric version 6.4 ou ultérieure. Consultez cet [article](service-fabric-cluster-creation-via-arm.md) pour la procédure de création d’un cluster Service Fabric à l’aide du modèle de ressource Azure.
* Certificat X.509 pour le chiffrement des secrets nécessaire pour se connecter au stockage pour stocker les sauvegardes. Consultez [l’article](service-fabric-cluster-creation-via-arm.md) pour savoir comment obtenir ou créer un certificat X.509.
* Application avec état fiable Service Fabric générée avec le kit SDK Service Fabric version 3.0 ou ultérieure. Pour les applications qui ciblent .NET Core 2.0, l’application doit être générée à l’aide du SDK Service Fabric version 3.1 ou ultérieure.
* Créez un compte de stockage Azure pour stocker les sauvegardes de l’application.
* Installez le module Microsoft.ServiceFabric.Powershell.Http [en préversion] pour effectuer des appels de configuration.

```powershell
    Install-Module -Name Microsoft.ServiceFabric.Powershell.Http -AllowPrerelease
```

* Assurez-vous que le cluster est connecté à l’aide de la commande `Connect-SFCluster` avant d’effectuer toute requête de configuration à l’aide du module Microsoft.ServiceFabric.Powershell.Http.

```powershell

    Connect-SFCluster -ConnectionEndpoint 'https://mysfcluster.southcentralus.cloudapp.azure.com:19080'   -X509Credential -FindType FindByThumbprint -FindValue '1b7ebe2174649c45474a4819dafae956712c31d3' -StoreLocation 'CurrentUser' -StoreName 'My' -ServerCertThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'  

```

## <a name="enabling-backup-and-restore-service"></a>Activation du service de sauvegarde et de restauration

### <a name="using-azure-portal"></a>En passant par le portail Azure

Cochez la case `Include backup restore service` sous `+ Show optional settings` dans l’onglet `Cluster Configuration`.

![Cocher Inclure le service de sauvegarde et restauration avec le portail][1]


### <a name="using-azure-resource-manager-template"></a>Utilisation d’un modèle Azure Resource Manager
Vous devez d’abord activer le _service de sauvegarde et de restauration_ dans votre cluster. Récupérez le modèle approprié pour le cluster que vous souhaitez déployer. Vous pouvez soit utiliser les [exemples de modèles](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype), soit créer un modèle Resource Manager. Activez le _service de sauvegarde et de restauration_ en effectuant les étapes suivantes :

1. Vérifiez si `apiversion` a la valeur **`2018-02-01`** pour la ressource `Microsoft.ServiceFabric/clusters` et, si ce n’est pas le cas, procédez à une mise à jour, comme indiqué dans l’extrait de code suivant :

    ```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. À présent, activez le _service de sauvegarde et de restauration_ en ajoutant la section `addonFeatures` suivante sous la section `properties`, comme indiqué dans l’extrait de code ci-après : 

    ```json
        "properties": {
            ...
            "addonFeatures":  ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```
3. Configurez un certificat X.509 pour le chiffrement des informations d’identification. Ceci est important afin de vous assurer que les informations d’identification fournies pour vous connecter au stockage sont chiffrées avant d’être rendues persistantes. Configurez le certificat de chiffrement en ajoutant la section `BackupRestoreService` suivante sous la section `fabricSettings`, comme indiqué dans l’extrait de code ci-après : 

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

4. Après avoir mis à jour votre modèle de cluster avec les modifications précédentes, appliquez-les et laissez le déploiement/la mise à niveau s’accomplir. Une fois l’opération terminée, le _service de sauvegarde et de restauration_ commence à s’exécuter dans votre cluster. L’URI de ce service est `fabric:/System/BackupRestoreService` et le service peut être situé sous la section du service système dans Service Fabric Explorer. 

## <a name="enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors"></a>Activation de la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors)
Examinons la procédure pour activer la sauvegarde périodique pour le service avec état fiable et les acteurs fiables (Reliable Actors). Cette procédure suppose les éléments suivants :
- Le cluster est configuré à l’aide de la sécurité X.509 avec le _service de sauvegarde et restauration_.
- Un service avec état fiable est déployé sur le cluster. Pour les besoins de ce guide de démarrage rapide, l’URI de l’application est `fabric:/SampleApp` et l’URI du service avec état fiable appartenant à cette application est `fabric:/SampleApp/MyStatefulService`. Ce service est déployé avec une partition unique et l’ID de partition est `974bd92a-b395-4631-8a7f-53bd4ae9cf22`.
- Le certificat client avec le rôle d’administrateur est installé dans le magasin _My_ (_Personal_) de l’emplacement du magasin de certificats _CurrentUser_ sur l’ordinateur à partir d’où les scripts ci-dessous sont appelés. Cet exemple utilise `1b7ebe2174649c45474a4819dafae956712c31d3` comme empreinte numérique de ce certificat. Pour plus d’informations sur les certificats clients, consultez [Contrôle d’accès en fonction du rôle pour les clients de Service Fabric](service-fabric-cluster-security-roles.md).

### <a name="create-backup-policy"></a>Créer la stratégie de sauvegarde

La première étape consiste à créer la stratégie de sauvegarde qui décrit la planification de la sauvegarde, le stockage cible pour les données de sauvegarde, le nom de la stratégie, les sauvegardes incrémentielles maximales autorisées avant le déclenchement de la sauvegarde complète et la stratégie de conservation pour le stockage des sauvegardes. 

Pour le stockage de sauvegarde, utilisez le compte de stockage Azure créé ci-dessus. Le conteneur `backup-container` est configuré pour stocker des sauvegardes. Lors du téléchargement de la sauvegarde, un conteneur portant ce nom est créé s’il n’existe pas déjà. Remplissez `ConnectionString` avec une chaîne de connexion valide pour le compte de stockage Azure, en remplaçant `account-name` par le nom de votre compte de stockage et `account-key` par la clé de votre compte de stockage.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

Exécutez les applets de commande PowerShell suivantes pour la création d’une stratégie de sauvegarde. Remplacez `account-name` par le nom de votre compte de stockage et `account-key` par la clé de votre compte de stockage.

```powershell

New-SFBackupPolicy -Name 'BackupPolicy1' -AutoRestoreOnDataLoss $true -MaxIncrementalBackups 20 -FrequencyBased -Interval 00:15:00 -AzureBlobStore -ConnectionString 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' -ContainerName 'backup-container' -Basic -RetentionDuration '10.00:00:00'

```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Exécutez le script PowerShell suivant pour appeler l’API REST requise afin de créer une stratégie. Remplacez `account-name` par le nom de votre compte de stockage et `account-key` par la clé de votre compte de stockage.

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$ScheduleInfo = @{
    Interval = 'PT15M'
    ScheduleKind = 'FrequencyBased'
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
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

```

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

1. Dans Service Fabric Explorer, accédez à l’onglet Sauvegardes et sélectionnez Actions > Créer une stratégie de sauvegarde.

    ![Créer la stratégie de sauvegarde][6]

2. Renseignez les informations. Pour les clusters Azure, AzureBlobStore doit être sélectionné.

    ![Créer une stratégie de sauvegarde du stockage Blob Azure][7]

### <a name="enable-periodic-backup"></a>Activer la sauvegarde périodique
Après avoir défini la stratégie de sauvegarde pour répondre aux exigences de protection des données de l’application, la stratégie doit être associée à l’application. Selon les besoins, la stratégie de sauvegarde peut être associée à une application, un service ou une partition.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

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
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/EnableBackup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
``` 

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

1. Sélectionnez une application et accédez à Action. Cliquez sur Activer/Mettre à jour la sauvegarde de l’application.

    ![Activer la sauvegarde de l’application][3]

2. Enfin, sélectionnez la stratégie souhaitée, puis cliquez sur Activer la sauvegarde.

    ![Sélectionner une stratégie][4]


### <a name="verify-that-periodic-backups-are-working"></a>Vérifier le fonctionnement des sauvegardes périodiques

Après avoir activé la sauvegarde au niveau de l’application, toutes les partitions appartenant à des services avec état fiables et des acteurs fiables (Reliable Actors) sous l’application commencent à être sauvegardées régulièrement conformément à la stratégie de sauvegarde associée. 

![Événement d’intégrité de partition sauvegardée][0]

### <a name="list-backups"></a>Répertorier les sauvegardes

Les sauvegardes associées à toutes les partitions appartenant à des services avec état fiables et des acteurs fiables (Reliable Actors) de l’application peuvent être énumérées à l’aide de l’API _GetBackups_. Les sauvegardes peuvent être énumérées pour une application, un service ou une partition.

#### <a name="powershell-using-microsoftservicefabricpowershellhttp-module"></a>PowerShell avec le module Microsoft.ServiceFabric.Powershell.Http

```powershell
    
Get-SFApplicationBackupList -ApplicationId WordCount
```

#### <a name="rest-call-using-powershell"></a>Appel REST avec PowerShell

Exécutez le script PowerShell suivant pour appeler l’API HTTP afin d’énumérer les sauvegardes créées pour toutes les partitions à l’intérieur de l’application `SampleApp`.

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Applications/SampleApp/$/GetBackups?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'

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

#### <a name="using-service-fabric-explorer"></a>Utiliser Service Fabric Explorer

Pour afficher les sauvegardes dans Service Fabric Explorer, accédez à une partition et sélectionnez l’onglet Sauvegardes.

![Énumérer les sauvegardes][5]

## <a name="limitation-caveats"></a>Limitations/mises en garde
- Les applets de commande PowerShell Service Fabric sont en mode Aperçu.
- Pas de prise en charge des clusters Service Fabric sur Linux.

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de la configuration de la sauvegarde périodique](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [Informations de référence sur l’API REST de sauvegarde et restauration](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/partition-backedup-health-event-azure.png
[1]: ./media/service-fabric-backuprestoreservice/enable-backup-restore-service-with-portal.png
[3]: ./media/service-fabric-backuprestoreservice/enable-app-backup.png
[4]: ./media/service-fabric-backuprestoreservice/enable-application-backup.png
[5]: ./media/service-fabric-backuprestoreservice/backup-enumeration.png
[6]: ./media/service-fabric-backuprestoreservice/create-bp.png
[7]: ./media/service-fabric-backuprestoreservice/creation-bp.png
