---
title: Présentation de la configuration de la sauvegarde périodique
description: Utilisez la fonctionnalité de sauvegarde et de restauration périodiques de Service Fabric pour configurer la sauvegarde périodique de vos services avec état fiable ou Reliable Actors.
ms.topic: article
ms.date: 2/01/2019
ms.openlocfilehash: 2607502af44b178131820d78f23bcdf4e32454a0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018883"
---
# <a name="understanding-periodic-backup-configuration-in-azure-service-fabric"></a>Compréhension de la configuration de la sauvegarde périodique dans Azure Service Fabric

La configuration de la sauvegarde périodique de vos Services fiables avec état ou Reliable Actors comprend les étapes suivantes :

1. **Création des stratégies de sauvegarde** : dans cette étape, une ou plusieurs stratégies de sauvegarde sont créées en fonction des besoins.

2. **Activation de la sauvegarde** : dans cette étape, vous associez les stratégies de sauvegarde créées à l’**étape 1** aux entités requises : une _application_, un _service_ ou une _partition_.

## <a name="create-backup-policy"></a>Créer la stratégie de sauvegarde

Une stratégie de sauvegarde se compose des configurations suivantes :

* **Restauration automatique en cas de perte de données** : spécifie s’il faut déclencher la restauration automatiquement à l’aide de la dernière sauvegarde disponible si la partition subit une perte de données.
> [!NOTE]
> Il est recommandé de ne PAS définir la restauration automatique dans les clusters de production
>

* **Nombre maximal de sauvegardes incrémentielles** : définit le nombre maximal de sauvegardes incrémentielles à effectuer entre deux sauvegardes complètes. Le nombre maximal de sauvegardes incrémentielles spécifie la limite supérieure. Une sauvegarde complète peut-être effectuée avant que le nombre spécifié de sauvegardes incrémentielles soit atteint dans l’une des situations suivantes

    1. Le réplica n’a jamais fait l’objet d’une sauvegarde complète depuis qu’il est devenu principal.

    2. Certains enregistrements du journal écrits depuis la dernière sauvegarde ont été tronqués.

    3. Le réplica a dépassé la limite MaxAccumulatedBackupLogSizeInMB.

* **Planification de la sauvegarde** : heure ou fréquence auxquelles effectuer les sauvegardes périodiques. Il est possible de planifier des sauvegardes périodiques à intervalles définis ou à heure fixe chaque jour ou semaine.

    1. **Planification de sauvegarde basée sur la fréquence** : ce type de planification doit être utilisé si les sauvegardes doivent être effectuées à intervalles fixes. L’intervalle de temps souhaité entre deux sauvegardes consécutives est défini à l’aide du format ISO8601. La planification de sauvegarde basée sur la fréquence prend en charge une résolution d’intervalle à la minute.
        ```json
        {
            "ScheduleKind": "FrequencyBased",
            "Interval": "PT10M"
        }
        ```

    2. **Planification de sauvegarde basée sur l’heure** : ce type de planification doit être utilisé si les sauvegardes doivent être effectuées à des heures spécifiques dans la journée ou la semaine. La fréquence de planification peut être quotidienne ou hebdomadaire.
        1. **_Quotidienne_ Planification de sauvegarde basée sur l’heure** : ce type de planification doit être utilisé si les sauvegardes doivent être effectuées à des heures spécifiques de la journée. Pour spécifier cela, définissez `ScheduleFrequencyType` sur _Quotidienne_, et `RunTimes` sur la liste des heures souhaitées pendant la journée au format ISO8601. La date spécifiée avec les heures sera ignorée. Par exemple, `0001-01-01T18:00:00` indique _06:00_ chaque jour, en ignorant la partie date _01-01-0001_. L’exemple ci-dessous illustre la configuration pour déclencher une sauvegarde quotidienne à _09:00_ et à _18:00_.

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Daily",
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

        2. **_Hebdomadaire_ Planification de sauvegarde basée sur l’heure** : ce type de planification doit être utilisé si les sauvegardes doivent être effectuées à des heures spécifiques de la journée. Pour spécifier cela, définissez `ScheduleFrequencyType` sur _Hebdomadaire_, et `RunDays` sur la liste des jours de la semaine durant lesquels une sauvegarde doit être déclenchée, et `RunTimes` sur la liste des heures souhaitées pendant la journée au format ISO8601. La date spécifiée avec les heures sera ignorée. Liste des jours de la semaine durant lesquels déclencher la sauvegarde périodique. L’exemple ci-dessous illustre la configuration pour déclencher une sauvegarde quotidienne à _09:00_ et à _18:00_, du lundi au vendredi.

            ```json
            {
                "ScheduleKind": "TimeBased",
                "ScheduleFrequencyType": "Weekly",
                "RunDays": [
                   "Monday",
                   "Tuesday",
                   "Wednesday",
                   "Thursday",
                   "Friday"
                ],
                "RunTimes": [
                  "0001-01-01T09:00:00Z",
                  "0001-01-01T18:00:00Z"
                ]
            }
            ```

* **Stockage de sauvegarde** : spécifie l’emplacement où charger les sauvegardes. Le stockage peut être un stockage d’objets blob Azure ou un partage de fichiers.
    1. **Stockage Blob Azure** : ce type de stockage doit être sélectionné si les sauvegardes générées dans Azure doivent être stockées. Des clusters _autonomes_ et _basés sur Azure_ peuvent utiliser ce type de stockage. La description de ce type de stockage nécessite une chaîne de connexion et le nom du conteneur dans lequel les sauvegardes doivent être chargées. Si le conteneur du nom spécifié n’est pas disponible, il est créé lors du chargement de la sauvegarde.

        ```json
        {
            "StorageKind": "AzureBlobStore",
            "FriendlyName": "Azure_storagesample",
            "ConnectionString": "<Put your Azure blob store connection string here>",
            "ContainerName": "BackupContainer"
        }
        ```

        > [!NOTE]
        > Le service de sauvegarde et de restauration ne fonctionne pas avec Stockage Azure v1.
        >

    2. **Partage de fichiers** : ce type de stockage doit être sélectionné pour les clusters _autonomes_ si la sauvegarde de données doit être stockée localement. La description de ce type de stockage requiert la spécification du chemin du partage de fichiers dans lequel les sauvegardes doivent être chargées. L’accès au partage de fichiers peut être configuré à l’aide d’une des options suivantes
        1. _Authentification Windows intégrée_ : le partage de fichiers est accessible à tous les ordinateurs appartenant au cluster Service Fabric. Dans ce cas, définissez les champs suivants pour configurer un stockage de sauvegarde basé sur un _partage de fichiers_.

            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore"
            }
            ```

        2. _Protection du partage de fichiers à l’aide d’un nom d’utilisateur et d’un mot de passe_ : le partage de fichiers est accessible à des utilisateurs spécifiques. La spécification du stockage du partage de fichiers offre également la possibilité de spécifier un nom d’utilisateur et un mot de passe secondaires pour fournir des informations d’identification de secours en cas d’échec de l’authentification avec le nom d’utilisateur et le mot de passe principaux. Dans ce cas, définissez les champs suivants pour configurer un stockage de sauvegarde basé sur un _partage de fichiers_.

            ```json
            {
                "StorageKind": "FileShare",
                "FriendlyName": "Sample_FileShare",
                "Path": "\\\\StorageServer\\BackupStore",
                "PrimaryUserName": "backupaccount",
                "PrimaryPassword": "<Password for backupaccount>",
                "SecondaryUserName": "backupaccount2",
                "SecondaryPassword": "<Password for backupaccount2>"
            }
            ```

> [!NOTE]
> Assurez-vous que la fiabilité du stockage correspond au minimum aux exigences de fiabilité des données de sauvegarde.
>

* **Stratégie de conservation** : spécifie la stratégie permettant de conserver les sauvegardes dans le stockage configuré. Seule la stratégie de conservation de base est prise en charge.
    1. **Stratégie de conservation de base** : cette stratégie de conservation permet de garantir une utilisation optimale du stockage en supprimant les fichiers de sauvegarde qui ne sont plus obligatoires. `RetentionDuration` peut être spécifié pour définir l’intervalle de temps pendant lequel les sauvegardes doivent être conservées dans le stockage. `MinimumNumberOfBackups` est un paramètre facultatif qui peut être spécifié pour vous assurer que le nombre spécifié de sauvegardes sont toujours conservées, indépendamment de `RetentionDuration`. L’exemple ci-dessous illustre la configuration de la conservation des sauvegardes pendant _10_ jours et n’autorise pas le nombre de sauvegardes à descendre en dessous de _20_.

        ```json
        {
            "RetentionPolicyType": "Basic",
            "RetentionDuration" : "P10D",
            "MinimumNumberOfBackups": 20
        }
        ```

## <a name="enable-periodic-backup"></a>Activer la sauvegarde périodique
Après définition d’une stratégie de sauvegarde répondant aux exigences de sauvegarde de données, cette stratégie doit être correctement associée à une _application_, à un _service_ ou à une _partition_.

> [!NOTE]
> Assurez-vous qu’aucune mise à niveau d’application n’est en cours avant d’activer la sauvegarde.
>

### <a name="hierarchical-propagation-of-backup-policy"></a>Propagation hiérarchique de la stratégie de sauvegarde
Dans Service Fabric, la relation entre l’application, le service et les partitions est hiérarchique, comme expliqué dans [Modèle d’application](./service-fabric-application-model.md). Une stratégie de sauvegarde peut être associée à une _application_, à un _service_, ou à une _partition_ dans la hiérarchie. La stratégie de sauvegarde se propage de façon hiérarchique au niveau suivant. En supposant qu’il n’y ait qu’une seule stratégie de sauvegarde créée et associée à une _application_, toutes les partitions avec état appartenant à tous les _Services fiables avec état_ et _Reliable Actors_ de l’_application_ seront sauvegardées à l’aide de la stratégie de sauvegarde. Ou bien, si la stratégie de sauvegarde est associée à un _Service fiable avec état_, toutes ses partitions seront sauvegardées à l’aide de la stratégie de sauvegarde.

### <a name="overriding-backup-policy"></a>Remplacement de stratégie de sauvegarde
Il peut y avoir un scénario où une sauvegarde de données avec la même planification de sauvegarde est requise pour tous les services de l’application, à l’exception de services spécifiques où il est nécessaire de disposer d’une sauvegarde de données utilisant une planification de fréquence supérieure, ou d’effectuer la sauvegarde vers un compte de stockage ou un partage de fichiers différents. Pour ces scénarios, le service de sauvegarde/restauration offre la possibilité de remplacer la stratégie propagée au niveau d’un service et d’une partition. Quand la stratégie de sauvegarde est associée à un _service_ ou à une _partition_, elle remplace une éventuelle stratégie de sauvegarde propagée.

### <a name="example"></a>Exemple

Cet exemple utilise une configuration avec deux applications, _MyApp_A_ et _MyApp_B_. L’application _MyApp_A_ contient deux services fiables avec état, _SvcA1_ & _SvcA3_, et un service Reliable Actor, _ActorA2_. Le service _SvcA1_ contient trois partitions, tandis que les services _ActorA2_ et _SvcA3_ contiennent chacun deux partitions.  L’application _MyApp_B_ contient trois services fiables avec état, _SvcB1_, _SvcB2_, et _SvcB3_. Les services _SvcB1_ et _SvcB2_ contiennent chacun deux partitions, tandis que le service _SvcB3_ en contient trois.

Supposons que les exigences de sauvegarde de données de ces applications sont les suivantes

1. MyApp_A
    1. Créer une sauvegarde quotidienne des données de toutes les partitions de tous les _Services fiables avec état_ et _Reliable Actors_ appartenant à l’application. Charger les données de sauvegarde dans l’emplacement _BackupStore1_.

    2. L’un des services, _SvcA3_, nécessite une sauvegarde de données toutes les heures.

    3. La taille des données de la partition _SvcA1_P2_ est plus importante que prévu, et ses données de sauvegarde doivent être stockées dans un autre emplacement de stockage, _BackupStore2_.

2. MyApp_B
    1. Créer une sauvegarde des données de toutes les partitions du service _SvcB1_ chaque dimanche à 8 h 00. Charger les données de sauvegarde dans l’emplacement _BackupStore1_.

    2. Créer une sauvegarde des données de la partition _SvcB2_P1_ chaque jour à 8 h 00. Charger les données de sauvegarde dans l’emplacement _BackupStore1_.

Pour répondre à ces exigences de sauvegarde de données, des stratégies de sauvegarde BP_1 à BP_5 sont créées, et la sauvegarde est activée comme suit.
1. MyApp_A
    1. Créer une stratégie de sauvegarde, _BP_1_, avec une planification de sauvegarde basée sur la fréquence, où la fréquence est définie sur 24 h. Le stockage de sauvegarde est configuré pour utiliser l’emplacement de stockage _BackupStore1_. Activer cette stratégie pour l’application _MyApp_A_ à l’aide de l’API [Activer la sauvegarde de l’application](/rest/api/servicefabric/sfclient-api-enableapplicationbackup). Cette action active la sauvegarde de données à l’aide de la stratégie de sauvegarde _BP_1_ pour toutes les partitions de _Services fiables avec état_ et de _Reliable Actors_ appartenant à l’application _MyApp_A_.

    2. Créer une stratégie de sauvegarde, _BP_2_, avec une planification de sauvegarde basée sur la fréquence, où la fréquence est définie sur 1 h. Le stockage de sauvegarde est configuré pour utiliser l’emplacement de stockage _BackupStore1_. Activer cette stratégie pour le service _SvcA3_ à l’aide de l’API [Activer la sauvegarde du service](/rest/api/servicefabric/sfclient-api-enableservicebackup). Cette action remplace la stratégie propagée _BP_1_ par la stratégie de sauvegarde explicitement activée _BP_2_ pour toutes les partitions du service _SvcA3_, ce qui amène la sauvegarde de données à utiliser la stratégie de sauvegarde _BP_2_ pour ces partitions.

    3. Créer une stratégie de sauvegarde, _BP_3_, avec une planification de sauvegarde basée sur la fréquence, où la fréquence est définie sur 24 h. Le stockage de sauvegarde est configuré pour utiliser l’emplacement de stockage _BackupStore2_. Activer cette stratégie pour la partition _SvcA1_P2_ à l’aide de L’API [Activer la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-enablepartitionbackup). Cette action remplace la stratégie propagée _BP_1_ par la stratégie de sauvegarde explicitement activée _BP_3_ pour la partition _SvcA1_P2_.

2. MyApp_B
    1. Créer une stratégie de sauvegarde, _BP_4_, avec une planification de sauvegarde basée sur l’heure, où le type de fréquence de planification est défini sur hebdomadaire, le jour d’exécution défini sur dimanche, et l’heure d’exécution définie sur 8 h 00. Le stockage de sauvegarde est configuré pour utiliser l’emplacement de stockage _BackupStore1_. Activer cette stratégie pour le service _SvcB1_ à l’aide de l’API [Activer la sauvegarde du service](/rest/api/servicefabric/sfclient-api-enableservicebackup). Cette action active la sauvegarde de données à l’aide de la stratégie de sauvegarde _BP_4_ pour toutes les partitions du service _SvcB1_.

    2. Créer une stratégie de sauvegarde, _BP_5_, avec une planification de sauvegarde basée sur l’heure, où le type de fréquence de planification est défini sur quotidien, et l’heure d’exécution définie sur 8 h 00. Le stockage de sauvegarde est configuré pour utiliser l’emplacement de stockage _BackupStore1_. Activer cette stratégie pour la partition _SvcB2_P1_ à l’aide de L’API [Activer la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-enablepartitionbackup). Cette action active la sauvegarde de données à l’aide de la stratégie de sauvegarde _BP_5_ pour la partition _SvcB2_P1_.

Le diagramme suivant illustre explicitement les stratégies de sauvegarde activées et les stratégies de sauvegarde propagées.

![Hiérarchie des applications Service Fabric][0]

## <a name="disable-backup"></a>Désactiver la sauvegarde
Les stratégies de sauvegarde peuvent être désactivées quand il n’est pas nécessaire de sauvegarder des données. Une stratégie de sauvegarde activée au niveau d’une _application_ ne peut être désactivée que pour cette _application_ à l’aide de l’API [Désactiver la sauvegarde de l’application](/rest/api/servicefabric/sfclient-api-disableapplicationbackup). Une stratégie de sauvegarde activée au niveau d’un _service_ ne peut être désactivée que pour ce _service_ à l’aide de l’API [Désactiver la sauvegarde du service](/rest/api/servicefabric/sfclient-api-disableservicebackup). Et une stratégie de sauvegarde activée au niveau d’une _partition_ ne peut être désactivée que pour cette _partition_ à l’aide de l’API [Désactiver la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-disablepartitionbackup).

* La désactivation d’une stratégie de sauvegarde pour une _application_ arrête toutes les sauvegardes périodiques de données qui se produisent en raison de la propagation de la stratégie de sauvegarde aux partitions de service avec état fiable ou aux partitions Reliable Actor.

* La désactivation d’une stratégie de sauvegarde pour un _service_ arrête toutes les sauvegardes périodiques de données qui se produisent en raison de la propagation de cette stratégie de sauvegarde aux partitions du _service_.

* La désactivation d’une stratégie de sauvegarde pour une _partition_ arrête toute sauvegarde périodique des données se produisant en raison de la stratégie de sauvegarde définie au niveau de la partition.

* Lors de la désactivation de la sauvegarde d’une entité (application/service/partition), `CleanBackup` peut être défini sur _true_ pour supprimer toutes les sauvegardes dans le stockage configuré.
    ```json
    {
        "CleanBackup": true 
    }
    ```
> [!NOTE]
> Assurez-vous qu’aucune mise à niveau d’application n’est en cours avant de désactiver la sauvegarde.
>

## <a name="suspend--resume-backup"></a>Suspendre et reprendre une sauvegarde
Certaines situations peuvent exiger une suspension temporaire de la sauvegarde périodique des données. Dans ce cas, selon l’exigence, l’API Suspendre la sauvegarde peut être utilisée au niveau d’une _application_, d’un _service_ ou d’une _partition_. La suspension d’une sauvegarde périodique est transitive vers la sous-arborescence de la hiérarchie de l’application à partir du point où elle est appliquée. 

* Lorsque la suspension est appliquée à une _application_ à l’aide de l’API [Suspendre la sauvegarde de l’application](/rest/api/servicefabric/sfclient-api-suspendapplicationbackup), la sauvegarde périodique des données est suspendue pour l’ensemble des services et partitions sous cette application.

* Lorsque la suspension est appliquée à un _service_ à l’aide de l’API [Suspendre la sauvegarde du service](/rest/api/servicefabric/sfclient-api-suspendservicebackup), la sauvegarde périodique des données est suspendue pour toutes les partitions sous ce service.

* Lorsque la suspension est appliquée à une _partition_ à l’aide de l’API [Suspendre la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-suspendpartitionbackup), la sauvegarde périodique des données est suspendue cette partition.

Une fois la nécessité de suspension passée, la sauvegarde périodique des données peut être restaurée à l’aide de l’API Reprendre la sauvegarde appropriée. La sauvegarde périodique doit être reprise au même niveau d’_application_, de _service_ ou de _partition_ que celui auquel elle a été suspendue.

* Si une suspension a été appliquée au niveau d’une _application_, elle doit être reprise à l’aide l’API [Reprendre la sauvegarde de l’application](/rest/api/servicefabric/sfclient-api-resumeapplicationbackup). 

* Si une suspension a été appliquée au niveau d’un _service_, elle doit être reprise à l’aide l’API [Reprendre la sauvegarde du service](/rest/api/servicefabric/sfclient-api-resumeservicebackup).

* Si une suspension a été appliquée au niveau d’une _partition_, elle doit être reprise à l’aide l’API [Reprendre la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-resumepartitionbackup).

### <a name="difference-between-suspend-and-disable-backups"></a>Différence entre la suspension et la désactivation des sauvegardes
Vous ne devriez désactiver les sauvegardes que lorsqu’elles ne sont plus nécessaires pour une application, un service ou une partition spécifiques. Il est en fait possible de demander la désactivation de la sauvegarde avec le paramètre de nettoyage des sauvegardes, ce qui a pour effet de supprimer également toutes les sauvegardes existantes. Il convient cependant de recourir à une suspension quand on souhaite désactiver temporairement les sauvegardes, par exemple, quand le disque local est saturé ou lorsque le chargement de la sauvegarde échoue en raison d’un problème réseau connu. 

Si une désactivation peut être demandée uniquement à un niveau précédemment activé explicitement pour la sauvegarde, une suspension peut être appliquée à tout niveau actuellement activé pour la sauvegarde, directement ou par voie d'héritage ou de hiérarchie. Par exemple, si la sauvegarde est activée au niveau d’une application, il est possible de demander la désactivation uniquement au niveau de l’application. En revanche, la suspension peut être demandée au niveau d’une application, d’un service ou d’une partition sous cette application. 

## <a name="auto-restore-on-data-loss"></a>Restauration automatique en cas de perte de données
Une partition de service peut perdre des données en raison de défaillances inattendues. Par exemple, le disque de deux réplicas sur trois pour une partition (y compris le réplica principal) est endommagé ou effacé.

Quand Service Fabric détecte que la partition perd des données, il appelle la méthode d’interface `OnDataLossAsync` sur la partition, et attend que la partition effectue l’action requise pour sortir de la perte de données. Dans ce cas, si la stratégie de sauvegarde effective au niveau la partition a l’indicateur `AutoRestoreOnDataLoss` défini sur `true`, la restauration est déclenchée automatiquement à l’aide de la dernière sauvegarde disponible pour cette partition.

> [!NOTE]
> Il est recommandé de ne PAS définir la restauration automatique dans les clusters de production
>

## <a name="get-backup-configuration"></a>Obtenir la configuration de la sauvegarde
Des API distinctes sont disponibles pour obtenir des informations sur la configuration de la sauvegarde au niveau d’une _application_, d’un _service_ et d’une _partition_. Ces API sont respectivement [Obtenir les informations sur la configuration de la sauvegarde de l’application](/rest/api/servicefabric/sfclient-api-getapplicationbackupconfigurationinfo), [Obtenir les informations sur la configuration de la sauvegarde du service](/rest/api/servicefabric/sfclient-api-getservicebackupconfigurationinfo) et [Obtenir les informations sur la configuration de la sauvegarde de la partition](/rest/api/servicefabric/sfclient-api-getpartitionbackupconfigurationinfo). Essentiellement, ces API retournent la stratégie de sauvegarde applicable, le niveau auquel la stratégie de sauvegarde est appliquée, et des détails sur la suspension de la sauvegarde. Voici une brève description des résultats retournés par ces API.

- Informations sur la configuration de la sauvegarde de l’application : fournissent les détails de la stratégie de sauvegarde appliquée au niveau de l’application, et toutes les stratégies remplacées au niveau des services et partitions appartenant à l’application. Ces résultats incluent également les informations de suspension de l’application ainsi que de ses services et partitions.

- Informations sur la configuration de la sauvegarde du service : fournissent les détails de la stratégie de sauvegarde effective du service, l’étendue d’application de cette stratégie et toutes les stratégies remplacées au niveau de ses partitions. Ces résultats incluent également les informations de suspension du service et de ses partitions.

- Informations sur la configuration de la sauvegarde de la partition : fournissent les détails de la stratégie de sauvegarde effective de la partition, et l’étendue d’application de cette stratégie. Ces résultats incluent également les informations de suspension des partitions.

## <a name="list-available-backups"></a>Répertorier les sauvegardes disponibles

Les sauvegardes disponibles peuvent être répertoriées à l’aide de l’API Obtenir la liste des sauvegardes. Le résultat de l’appel de l’API inclut des éléments d’informations sur la sauvegarde, liés à toutes les sauvegardes disponibles sur le stockage de sauvegarde configuré dans la stratégie de sauvegarde applicable. Différentes variantes de cette API sont fournies pour répertorier les sauvegardes disponibles appartenant à une application, à un service ou à une partition. Ces API prennent en charge l’obtention de la _dernière_ sauvegarde disponible de toutes les partitions applicables, ainsi que le filtrage des sauvegardes en fonction de la _date de début_ et de la _date de fin_.

Ces API prennent également en charge la pagination des résultats. Quand le paramètre _MaxResults_ est défini sur un entier positif différent de zéro, l’API retourne les éléments d’informations sur la sauvegarde _MaxResults_. Il peut arriver que des éléments d’informations sur la sauvegarde autres que la valeur _MaxResults_ soient disponibles. Un jeton de continuation est alors renvoyé. Un paramètre de jeton de continuation valide peut être utilisé pour obtenir le jeu de résultats suivant. Quand une valeur de jeton de continuation valide est transmise à l’appel suivant de l’API, celle-ci retourne le jeu de résultats suivant. Quand tous les résultats disponibles ont été retournés, aucun jeton de continuation n’est inclus dans la réponse.

Voici de brèves informations sur les variantes prises en charge.

- [Obtenir la liste des sauvegardes d’application](/rest/api/servicefabric/sfclient-api-getapplicationbackuplist) : retourne la liste des sauvegardes disponibles pour chaque partition appartenant à une application Service Fabric donnée.

- [Obtenir la liste des sauvegardes de service](/rest/api/servicefabric/sfclient-api-getservicebackuplist) : retourne la liste des sauvegardes disponibles pour chaque partition appartenant à un service Service Fabric donné.
 
- [Obtenir la liste des sauvegardes de partition](/rest/api/servicefabric/sfclient-api-getpartitionbackuplist) : retourne la liste des sauvegardes disponibles pour la partition spécifiée.

## <a name="next-steps"></a>Étapes suivantes
- [Informations de référence sur l’API REST de sauvegarde et restauration](/rest/api/servicefabric/sfclient-index-backuprestore)

[0]: ./media/service-fabric-backuprestoreservice/backup-policy-association-example.png
