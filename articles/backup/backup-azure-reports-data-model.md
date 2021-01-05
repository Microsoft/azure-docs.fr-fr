---
title: Modèle de données pour les événements de diagnostics de sauvegarde Azure
description: Ce modèle de données fait référence au mode spécifique à la ressource pour envoyer des événements de diagnostic à Log Analytics (LA).
ms.topic: conceptual
ms.date: 10/30/2019
ms.openlocfilehash: 52c5c0694ed59aea20453ae7a2bd3209d76df433
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92173971"
---
# <a name="data-model-for-azure-backup-diagnostics-events"></a>Modèle de données pour les événements de diagnostics de sauvegarde Azure

## <a name="coreazurebackup"></a>CoreAzureBackup

Ce tableau fournit des informations sur les entités de sauvegarde principales, comme les coffres et les éléments de sauvegarde.

| **Champ**                         | **Type de données** | **Description**                                              |
| --------------------------------- | ------------- | ------------------------------------------------------------ |
| ResourceId                        | Texte          | Identificateur de ressource des données collectées. Par exemple, ID de ressource du coffre Recovery Services. |
| NomOpération                     | Texte          | Ce champ représente le nom de l’opération en cours - BackupItem, BackupItemAssociation ou ProtectedContainer. |
| Category                          | Texte          | Ce champ représente la catégorie des données de diagnostic envoyées (push) aux journaux Azure Monitor. Par exemple, CoreAzureBackup. |
| AgentVersion                      | Texte          | Numéro de version de l’agent de sauvegarde ou de l’agent de protection (dans le cas de SC DPM et MABS) |
| AzureBackupAgentVersion           | Texte          | Version de l’agent Sauvegarde Azure sur le serveur de gestion des sauvegardes |
| AzureDataCenter                   | Texte          | Centre de données dans lequel se trouve le coffre                       |
| BackupItemAppVersion              | Texte          | Version de l’application de l’élément de sauvegarde                       |
| BackupItemFriendlyName            | Texte          | Nom convivial de l’élément de sauvegarde                             |
| BackupItemName                    | Texte          | Nom de l’élément de sauvegarde                                      |
| BackupItemProtectionState         | Texte          | État de protection de l’élément de sauvegarde                          |
| BackupItemFrontEndSize            | Texte          | Taille frontale de l’élément de sauvegarde                            |
| BackupItemType                    | Texte          | Type de l’élément de sauvegarde. Par exemple : Machine virtuelle, FileFolder             |
| BackupItemUniqueId                | Texte          | Identificateur unique de l’élément de sauvegarde                         |
| BackupManagementServerType        | Texte          | Type du serveur de gestion des sauvegardes, par exemple as MABS et SC DPM     |
| BackupManagementServerUniqueId    | Texte          | Champ qui identifie de façon unique le serveur de gestion des sauvegardes      |
| BackupManagementType              | Texte          | Type de fournisseur pour le serveur qui effectue le travail de sauvegarde. Par exemple, IaaSVM, FileFolder |
| BackupManagementServerName        | Texte          | Nom du serveur de gestion des sauvegardes                         |
| BackupManagementServerOSVersion   | Texte          | Version du système d’exploitation du serveur de gestion des sauvegardes                   |
| BackupManagementServerVersion     | Texte          | Version du serveur de gestion des sauvegardes                      |
| LatestRecoveryPointLocation       | Texte          | Emplacement du point de récupération le plus récent pour l’élément de sauvegarde    |
| LatestRecoveryPointTime           | DateTime      | Date et heure du point de récupération le plus récent pour l’élément de sauvegarde   |
| OldestRecoveryPointLocation       | Texte          | Emplacement du point de récupération le plus ancien pour l’élément de sauvegarde    |
| OldestRecoveryPointTime           | DateTime      | Date et heure du point de récupération le plus récent pour l’élément de sauvegarde   |
| PolicyUniqueId                    | Texte          | ID unique d’identification de la stratégie                             |
| ProtectedContainerFriendlyName    | Texte          | Nom convivial du serveur protégé                        |
| ProtectedContainerLocation        | Texte          | Indique si le conteneur protégé est situé en local ou dans Azure |
| ProtectedContainerName            | Texte          | Nom du conteneur protégé                            |
| ProtectedContainerOSType          | Texte          | Type de système d’exploitation du conteneur protégé                          |
| ProtectedContainerOSVersion       | Texte          | Version du système d’exploitation du conteneur protégé                        |
| ProtectedContainerProtectionState | Texte          | État de protection du conteneur protégé                  |
| ProtectedContainerType            | Texte          | Indique si le conteneur protégé est un serveur ou un conteneur  |
| ProtectedContainerUniqueId        | Texte          | ID unique qui identifie le conteneur protégé pour tout élément sauf les machines virtuelles sauvegardées à l’aide de DPM ou MABS |
| ProtectedContainerWorkloadType    | Texte          | Type du conteneur protégé sauvegardé. Par exemple, IaaSVMContainer |
| ProtectionGroupName               | Texte          | Nom du groupe de protection dans lequel l’élément de sauvegarde est protégé, pour SC DPM et MABS, le cas échéant |
| ResourceGroupName                 | Texte          | Groupe de ressources de la ressource (par exemple coffre Recovery Services) pour les données collectées |
| SchemaVersion                     | Texte          | Ce champ indique la version actuelle du schéma. Il s’agit de **V2** |
| SecondaryBackupProtectionState    | Texte          | Indique si la protection secondaire est activée pour l’élément de sauvegarde  |
| State                             | Texte          | État de l’objet élément de sauvegarde. Par exemple, Actif, Supprimé |
| StorageReplicationType            | Texte          | Type de réplication de stockage pour le coffre. Par exemple, Géoredondant |
| SubscriptionId                    | Texte          | Identificateur d’abonnement de la ressource (par exemple, coffre Recovery Services) pour laquelle les données sont collectées |
| VaultName                         | Texte          | Nom du coffre                                            |
| VaultTags                         | Texte          | Étiquettes associées à la ressource de coffre                    |
| VaultUniqueId                     | Texte          | Identificateur unique du coffre                             |
| SourceSystem                      | Texte          | Système source des données actuelles - Azure                  |

## <a name="addonazurebackupalerts"></a>AddonAzureBackupAlerts

Ce tableau fournit plus d’informations sur les champs liés aux alertes.

| **Champ**                      | **Type de données** | **Description**                                              |
| :----------------------------- | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texte          | Identificateur unique de la ressource sur laquelle les données sont collectées. Par exemple, un ID de ressource de coffre Recovery Services |
| NomOpération                  | Texte          | Nom de l’opération en cours. Par exemple, Alerte            |
| Category                       | Texte          | Catégorie de données de diagnostic envoyées (push) aux journaux Azure Monitor - AddonAzureBackupAlerts |
| AlertCode                      | Texte          | Code qui identifie un type d’alerte                     |
| AlertConsolidationStatus       | Texte          | Indique si l’alerte est une alerte consolidée ou non         |
| AlertOccurrenceDateTime        | DateTime      | Date et heure de la création de l’alerte                     |
| AlertRaisedOn                  | Texte          | Type d’entité sur lequel l’alerte est déclenchée                        |
| AlertSeverity                  | Texte          | Gravité de l’alerte. Par exemple, Critique                 |
| AlertStatus                    | Texte          | État de l’alerte. Par exemple, Actif                     |
| AlertTimeToResolveInMinutes    | Number        | Temps nécessaire pour résoudre une alerte. Vide pour les alertes actives.     |
| AlertType                      | Texte          | Type of alert. Par exemple, Sauvegarde                           |
| AlertUniqueId                  | Texte          | Identificateur unique de l’alerte générée                    |
| BackupItemUniqueId             | Texte          | Identificateur unique de l’élément de sauvegarde associé à l’alerte |
| BackupManagementServerUniqueId | Texte          | Champ qui identifie de manière unique le serveur de gestion des sauvegardes par le biais duquel l’élément de sauvegarde est protégé, le cas échéant |
| BackupManagementType           | Texte          | Type de fournisseur pour le serveur exécutant la sauvegarde, par exemple, Machine virtuelle IaaS, Dossier de fichiers |
| CountOfAlertsConsolidated      | Number        | Nombre d’alertes consolidées dans le cas d’une alerte consolidée  |
| ProtectedContainerUniqueId     | Texte          | Identificateur unique du serveur protégé associé à l’alerte |
| RecommendedAction              | Texte          | Action recommandée pour résoudre l’alerte                      |
| SchemaVersion                  | Texte          | Version actuelle du schéma, par exemple **V2**            |
| State                          | Texte          | État actuel de l’objet alerte, par exemple, Actif, Supprimé |
| StorageUniqueId                | Texte          | ID unique utilisé pour identifier l’entité de stockage                |
| VaultUniqueId                  | Texte          | ID unique utilisé pour identifier le coffre associé à l’alerte    |
| SourceSystem                   | Texte          | Système source des données actuelles - Azure                    |

## <a name="addonazurebackupprotectedinstance"></a>AddonAzureBackupProtectedInstance

Ce tableau indique les champs de base relatifs aux instances protégées.

| **Champ**                      | **Type de données** | **Description**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texte          | Identificateur unique de la ressource sur laquelle les données sont collectées. Par exemple, un ID de ressource de coffre Recovery Services |
| NomOpération                  | Texte          | Nom de l’opération, par exemple ProtectedInstance         |
| Category                       | Texte          | Catégorie de données de diagnostic envoyées (push) aux journaux Azure Monitor - AddonAzureBackupProtectedInstance |
| BackupItemUniqueId             | Texte          | ID unique de l’élément de sauvegarde                                 |
| BackupManagementServerUniqueId | Texte          | Champ qui identifie de manière unique le serveur de gestion des sauvegardes par le biais duquel l’élément de sauvegarde est protégé, le cas échéant |
| BackupManagementType           | Texte          | Type de fournisseur pour le serveur exécutant la sauvegarde, par exemple, Machine virtuelle IaaS, Dossier de fichiers |
| ProtectedContainerUniqueId     | Texte          | ID unique identifiant le conteneur protégé sur lequel le travail est exécuté |
| ProtectedInstanceCount         | Texte          | Nombre d’instances protégées pour le conteneur protégé ou l’élément de sauvegarde associé à ces date et heure |
| SchemaVersion                  | Texte          | Version actuelle du schéma, par exemple **V2**            |
| State                          | Texte          | État de l’objet élément de sauvegarde, par exemple, Actif, Supprimé |
| VaultUniqueId                  | Texte          | Identificateur unique du coffre protégé associé à l’instance protégée |
| SourceSystem                   | Texte          | Système source des données actuelles - Azure                    |

## <a name="addonazurebackupjobs"></a>AddonAzureBackupJobs

Ce tableau fournit plus d’informations sur les champs liés aux travaux.

| **Champ**                      | **Type de données** | **Description**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texte          | Identificateur de ressource des données collectées. Par exemple, ID de ressource du coffre Recovery Services |
| NomOpération                  | Texte          | Ce champ représente le nom de l’opération en cours - Travail    |
| Category                       | Texte          | Ce champ représente la catégorie des données de diagnostic envoyées (push) aux journaux Azure Monitor - AddonAzureBackupJobs |
| AdhocOrScheduledJob            | Texte          | Champ qui indique si le travail est de type ad hoc ou planifié           |
| BackupItemUniqueId             | Texte          | ID unique utilisé pour identifier l’élément de sauvegarde associé à l’entité de stockage |
| BackupManagementServerUniqueId | Texte          | ID unique utilisé pour identifier le serveur de gestion des sauvegardes associé à l’entité de stockage |
| BackupManagementType           | Texte          | Type de fournisseur exécutant la sauvegarde (par exemple, Machine virtuelle IaaS, Dossier de fichiers) auquel appartient ce travail |
| DataTransferredInMB            | Number        | Données transférées en Mo pour ce travail                          |
| JobDurationInSecs              | Number        | Durée totale de la tâche en secondes                                |
| JobFailureCode                 | Texte          | Chaîne de Code d’échec en raison du type d’échec de travail survenu    |
| JobOperation                   | Texte          | Opération pour laquelle le travail est exécuté, par exemple, Sauvegarde, Restauration, Configuration de sauvegarde |
| JobOperationSubType            | Texte          | Sous-type de l’opération de travail. Par exemple, « log », en cas de travail de sauvegarde du journal |
| JobStartDateTime               | DateTime      | Date et heure de démarrage de l’exécution du travail                       |
| JobStatus                      | Texte          | État du travail terminé par exemple, Terminé, Échec   |
| JobUniqueId                    | Texte          | ID unique d’identification du travail                                |
| ProtectedContainerUniqueId     | Texte          | Identificateur unique du serveur protégé associé au travail |
| RecoveryJobDestination         | Texte          | Destination d’un travail de récupération, où les données sont récupérées   |
| RecoveryJobRPDateTime          | DateTime      | Date et heure de la création du point de récupération en cours de récupération |
| RecoveryJobLocation            | Texte          | Emplacement auquel a été stocké le point de récupération en cours de récupération |
| RecoveryLocationType           | Texte          | Type de l’emplacement de récupération                                |
| SchemaVersion                  | Texte          | Version actuelle du schéma, par exemple **V2**            |
| State                          | Texte          | État actuel de l’objet travail, par exemple, « Actif » ou « Supprimé » |
| VaultUniqueId                  | Texte          | Identificateur unique du coffre protégé associé au travail |
| SourceSystem                   | Texte          | Système source des données actuelles - Azure                    |

## <a name="addonazurebackuppolicy"></a>AddonAzureBackupPolicy

Ce tableau fournit plus d’informations sur les champs liés à la stratégie.

| **Champ**                       | **Type de données**  | **Description**                                              |
| ------------------------------- | -------------- | ------------------------------------------------------------ |
| ResourceId                      | Texte           | Identificateur unique de la ressource sur laquelle les données sont collectées. Par exemple, un ID de ressource de coffre Recovery Services |
| NomOpération                   | Texte           | Nom de l’opération, par exemple, Policy ou PolicyAssociation |
| Category                        | Texte           | Catégorie de données de diagnostic envoyées (push) aux journaux Azure Monitor - AddonAzureBackupPolicy |
| BackupDaysOfTheWeek             | Texte           | Jours de la semaine auxquels les sauvegardes sont planifiées            |
| BackupFrequency                 | Texte           | Fréquence à laquelle les sauvegardes sont exécutées. Par exemple, Quotidienne, Hebdomadaire |
| BackupManagementType            | Texte           | Type de fournisseur pour le serveur qui effectue le travail de sauvegarde. Par exemple, IaaSVM, FileFolder |
| BackupManagementServerUniqueId  | Texte           | Champ qui identifie de manière unique le serveur de gestion des sauvegardes par le biais duquel l’élément de sauvegarde est protégé, le cas échéant |
| BackupTimes                     | Texte           | Date et heure auxquelles les sauvegardes sont planifiées                     |
| DailyRetentionDuration          | Nombre entier   | Durée totale de rétention en jours des sauvegardes configurées      |
| DailyRetentionTimes             | Texte           | Date et heure auxquelles la rétention quotidienne est configurée            |
| DiffBackupDaysOfTheWeek         | Texte           | Jours de la semaine où sont effectuées les sauvegardes différentielles pour SQL dans une sauvegarde de machine virtuelle Azure |
| DiffBackupFormat                | Texte           | Format des sauvegardes différentielles pour SQL dans une sauvegarde de machine virtuelle Azure   |
| DiffBackupRetentionDuration     | Nombre décimal | Durée de conservation des sauvegardes différentielles pour SQL dans une sauvegarde de machine virtuelle Azure |
| DiffBackupTime                  | Temps           | Durée des sauvegardes différentielles pour SQL dans une sauvegarde de machine virtuelle Azure     |
| LogBackupFrequency              | Nombre décimal | Fréquence des sauvegardes de fichier journal pour SQL                            |
| LogBackupRetentionDuration      | Nombre décimal | Durée de conservation des sauvegardes de fichier journal pour SQL dans une sauvegarde de machine virtuelle Azure |
| MonthlyRetentionDaysOfTheMonth  | Texte           | Semaines du mois au cours desquelles la rétention mensuelle est configurée.  Par exemple, Première, Dernière |
| MonthlyRetentionDaysOfTheWeek   | Texte           | Jours de la semaine sélectionnés pour la rétention mensuelle              |
| MonthlyRetentionDuration        | Texte           | Durée totale de rétention en mois des sauvegardes configurées    |
| MonthlyRetentionFormat          | Texte           | Type de configuration de la rétention mensuelle. Par exemple, quotidienne pour une configuration quotidienne, hebdomadaire pour une configuration hebdomadaire |
| MonthlyRetentionTimes           | Texte           | Date et heure auxquelles la rétention mensuelle est configurée           |
| MonthlyRetentionWeeksOfTheMonth | Texte           | Semaines du mois au cours desquelles la rétention mensuelle est configurée.   Par exemple, Première, Dernière |
| PolicyName                      | Texte           | Nom de la stratégie définie                                   |
| PolicyUniqueId                  | Texte           | ID unique d’identification de la stratégie                             |
| PolicyTimeZone                  | Texte           | Fuseau horaire dans lequel les champs d’heure de la stratégie sont spécifiés dans les journaux |
| RetentionDuration               | Texte           | Durée de rétention des sauvegardes configurées                    |
| RetentionType                   | Texte           | Type de rétention                                            |
| SchemaVersion                   | Texte           | Ce champ indique la version actuelle du schéma, **V2** |
| State                           | Texte           | État actuel de l’objet stratégie. Par exemple, Actif, Supprimé |
| SynchronisationFrequencyPerDay  | Nombre entier   | Nombre de synchronisations quotidiennes d’une sauvegarde de fichiers pour SC DPM et MABS |
| VaultUniqueId                   | Texte           | ID unique du coffre auquel cette stratégie appartient          |
| WeeklyRetentionDaysOfTheWeek    | Texte           | Jours de la semaine sélectionnés pour la rétention hebdomadaire               |
| WeeklyRetentionDuration         | Nombre décimal | Durée totale de rétention hebdomadaire en semaines des sauvegardes configurées |
| WeeklyRetentionTimes            | Texte           | Date et heure auxquelles la rétention hebdomadaire est configurée            |
| YearlyRetentionDaysOfTheMonth   | Texte           | Dates du mois sélectionnées pour la rétention annuelle             |
| YearlyRetentionDaysOfTheWeek    | Texte           | Jours de la semaine sélectionnés pour la rétention annuelle               |
| YearlyRetentionDuration         | Nombre décimal | Durée totale de rétention en années des sauvegardes configurées     |
| YearlyRetentionFormat           | Texte           | Type de configuration de la rétention annuelle, par exemple, quotidienne pour une configuration quotidienne, hebdomadaire pour une configuration hebdomadaire |
| YearlyRetentionMonthsOfTheYear  | Texte           | Mois de l’année sélectionnés pour la rétention annuelle             |
| YearlyRetentionTimes            | Texte           | Date et heure auxquelles la rétention annuelle est configurée            |
| YearlyRetentionWeeksOfTheMonth  | Texte           | Semaines du mois sélectionnées pour la rétention annuelle             |
| SourceSystem                    | Texte           | Système source des données actuelles - Azure                    |

## <a name="addonazurebackupstorage"></a>AddonAzureBackupStorage

Ce tableau fournit plus d’informations sur les champs liés au stockage.

| **Champ**                      | **Type de données** | **Description**                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------ |
| ResourceId                     | Texte          | Identificateur de ressource des données collectées. Par exemple, ID de ressource du coffre Recovery Services |
| NomOpération                  | Texte          | Ce champ représente le nom de l’opération en cours - Storage ou StorageAssociation |
| Category                       | Texte          | Ce champ représente la catégorie des données de diagnostic envoyées (push) aux journaux Azure Monitor - AddonAzureBackupStorage |
| BackupItemUniqueId             | Texte          | ID unique qui identifie l’élément de sauvegarde pour les machines virtuelles sauvegardées à l’aide de DPM ou MABS |
| BackupManagementServerUniqueId | Texte          | Champ qui identifie de manière unique le serveur de gestion des sauvegardes par le biais duquel l’élément de sauvegarde est protégé, le cas échéant |
| BackupManagementType           | Texte          | Type de fournisseur pour le serveur qui effectue le travail de sauvegarde. Par exemple, IaaSVM, FileFolder |
| PreferredWorkloadOnVolume      | Texte          | Les charges de travail pour lesquelles ce volume est le stockage par défaut      |
| ProtectedContainerUniqueId     | Texte          | Identificateur unique du conteneur protégé associé à l’élément de sauvegarde |
| SchemaVersion                  | Texte          | Version du schéma. Par exemple, **V2**                   |
| State                          | Texte          | État de l’objet élément de sauvegarde. Par exemple, Actif, Supprimé |
| StorageAllocatedInMBs          | Number        | Taille de stockage allouée par l’élément de sauvegarde correspondant dans le stockage correspondant de type disque |
| StorageConsumedInMBs           | Number        | Taille de stockage utilisée par l’élément de sauvegarde correspondant dans le stockage correspondant |
| StorageName                    | Texte          | Nom de l’entité de stockage. Par exemple, E:\                      |
| StorageTotalSizeInGBs          | Texte          | Taille totale du volume de stockage, en Go, consommé par l’entité de stockage     |
| StorageType                    | Texte          | Type de stockage, par exemple Cloud, Volume, Disque             |
| StorageUniqueId                | Texte          | ID unique utilisé pour identifier l’entité de stockage                |
| VaultUniqueId                  | Texte          | ID unique utilisé pour identifier le coffre associé à l’entité de stockage |
| VolumeFriendlyName             | Texte          | Nom convivial du volume de stockage                          |
| SourceSystem                   | Texte          | Système source des données actuelles - Azure                    |

## <a name="valid-operation-names-for-each-table"></a>Noms d’opération valides pour chaque table

Chaque enregistrement figurant dans les tables ci-dessus possède un **nom d’opération** associé. Un nom d’opération décrit le type d’enregistrement (et indique également les champs de la table qui sont remplis pour cet enregistrement). Chaque table (catégorie) prend en charge un ou plusieurs noms d’opération distincts. Vous trouverez ci-dessous un résumé des noms d’opération pris en charge pour chacune des tables ci-dessus.

| **Nom de la table / Catégorie**                   | **Nom d’opération pris en charge** | **Description**              |
| ------------------------------------------- | ------------------------------|----------------------------- |
| CoreAzureBackup | BackupItem | Représente un enregistrement contenant tous les détails d’un élément de sauvegarde donné, tels que l’ID, le nom, le type, etc. |
| CoreAzureBackup | BackupItemAssociation | Représente un mappage entre un élément de sauvegarde et son conteneur protégé associé (le cas échéant). |
| CoreAzureBackup | BackupItemFrontEndSizeConsumption | Représente un mappage entre un élément de sauvegarde et sa taille de front-end. |
| CoreAzureBackup | ProtectedContainer | Représente un enregistrement contenant tous les détails d’un conteneur protégé donné, tels que l’ID, le nom, le type, etc. |
| CoreAzureBackup | ProtectedContainerAssociation | Représente un mappage entre un conteneur protégé et le coffre utilisé pour sa sauvegarde. |
| CoreAzureBackup | Coffre | Représente un enregistrement contenant tous les détails d’un coffre donné, tels que l’ID, le nom, les étiquettes, l’emplacement, etc. |
| CoreAzureBackup | RecoveryPoint | Représente un enregistrement contenant le point de récupération le plus ancien et le dernier point de récupération pour un élément de sauvegarde donné. |
| AddonAzureBackupJobs | Travail |  Représente un enregistrement contenant tous les détails d’un travail donné. Par exemple, l’opération de travail, l’heure de début, l’état, etc. |
| AddonAzureBackupAlerts | Alerte | Représente un enregistrement contenant tous les détails d’une alerte donnée. Par exemple, l’heure de création de l’alerte, la gravité, l’état, etc.  |
| AddonAzureBackupStorage | Stockage | Représente un enregistrement contenant tous les détails d’une entité de stockage donnée. Par exemple, le nom de stockage, le type, etc. |
| AddonAzureBackupStorage | StorageAssociation | Représente un mappage entre un élément de sauvegarde et le stockage cloud total consommé par cet élément de sauvegarde. |
| AddonAzureBackupProtectedInstance | ProtectedInstance | Représente un enregistrement contenant le nombre d’instances protégées pour chaque conteneur ou élément de sauvegarde. Pour la sauvegarde des machines virtuelles Azure, le nombre d’instances protégées est disponible au niveau de l’élément de sauvegarde. Pour les autres charges de travail, il est disponible au niveau du conteneur protégé. |
| AddonAzureBackupPolicy | Stratégie |  Représente un enregistrement contenant tous les détails d’une stratégie de sauvegarde et de conservation. Par exemple, l’ID, le nom, les paramètres de conservation, etc. |
| AddonAzureBackupPolicy | PolicyAssociation | Représente un mappage entre un élément de sauvegarde et la stratégie de sauvegarde qui lui est appliquée. |   

Souvent, vous devez effectuer des jointures entre des tables différentes, ainsi que des jeux d’enregistrements différents qui font partie de la même table (différenciés par le nom d’opération), afin d’obtenir tous les champs requis pour votre analyse. Reportez-vous aux [exemples de requêtes](./backup-azure-monitoring-use-azuremonitor.md#sample-kusto-queries) pour démarrer. 

## <a name="next-steps"></a>Étapes suivantes

- [Découvrez comment envoyer des données de diagnostic à Log Analytics](./backup-azure-diagnostic-events.md)
- [Découvrez comment écrire des requêtes sur des tables spécifiques à des ressources](./backup-azure-monitoring-use-azuremonitor.md#sample-kusto-queries)