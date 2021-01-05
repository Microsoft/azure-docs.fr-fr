---
title: surveiller la sauvegarde Azure avec Azure Monitor
description: Supervisez les charges de travail de Sauvegarde Azure et créez des alertes personnalisées avec Azure Monitor.
ms.topic: conceptual
ms.date: 06/04/2019
ms.assetid: 01169af5-7eb0-4cb0-bbdb-c58ac71bf48b
ms.openlocfilehash: 3f5f663a2f0ed0f91cc414d352e975a2ff3b9649
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88827152"
---
# <a name="monitor-at-scale-by-using-azure-monitor"></a>Superviser à grande échelle avec Azure Monitor

Sauvegarde Azure fournit [des fonctionnalités intégrées de supervision et d’alerte](backup-azure-monitoring-built-in-monitor.md) dans un coffre Recovery Services. Ces fonctionnalités sont disponibles sans infrastructure de gestion supplémentaire. Ce service intégré est cependant limité aux scénarios suivants :

- Si vous supervisez des données de plusieurs coffres Recovery Services sur différents abonnements
- Si le canal de notification favori n’est *pas* l’e-mail
- Si les utilisateurs veulent des alertes pour d’autres scénarios
- Si vous voulez voir des informations provenant de composants locaux comme System Center Data Protection Manager (SC-DPM) dans Azure qui ne sont pas visibles dans [**Travaux de sauvegarde**](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault) ou dans [**Alertes de sauvegarde**](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault)

## <a name="using-log-analytics-workspace"></a>Utilisation d’un espace de travail Log Analytics

### <a name="create-alerts-by-using-log-analytics"></a>Créer des alertes avec Log Analytics

Dans Azure Monitor, vous pouvez créer vos propres alertes dans un espace de travail Log Analytics. Dans l’espace de travail, vous utilisez des *groupes d’actions Azure* pour sélectionner votre mécanisme de notification préféré.

> [!IMPORTANT]
> Pour plus d’informations sur le coût de création de cette requête, consultez [Tarification d’Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Ouvrez la section **Journaux** de l’espace de travail Log Analytics et créez une requête pour vos propres journaux. Quand vous sélectionnez **Nouvelle règle d’alerte**, la page de création d’alerte Azure Monitor s’ouvre, comme illustré dans l’image suivante.

![Créer une alerte dans un espace de travail Log Analytics](media/backup-azure-monitoring-laworkspace/custom-alert.png)

Ici, la ressource est déjà marquée, car l’espace de travail Log Analytics et l’intégration des groupes d’actions sont fournis.

![Page de création d’alerte Log Analytics](media/backup-azure-monitoring-laworkspace/inkedla-azurebackup-createalert.jpg)

#### <a name="alert-condition"></a>Condition d’alerte

La caractéristique définissant une alerte est sa condition de déclenchement. Sélectionnez **Condition** pour charger automatiquement la requête Kusto dans la page **Journaux**, comme illustré dans l’image suivante. Ici, vous pouvez modifier la condition pour l’adapter à vos besoins. Pour plus d’informations, consultez [Exemples de requêtes Kusto](#sample-kusto-queries).

![Configuration d’une condition d’alerte](media/backup-azure-monitoring-laworkspace/la-azurebackup-alertlogic.png)

Si nécessaire, vous pouvez modifier la requête Kusto. Choisissez un seuil, une période et une fréquence. Le seuil détermine à quel moment l’alerte sera déclenchée. La période correspond à la fenêtre de temps pendant laquelle la requête est exécutée. Par exemple, si le seuil est supérieur à 0, la période est de 5 minutes et la fréquence est de 5 minutes, la règle exécute la requête toutes les 5 minutes, en examinant les 5 minutes précédentes. Si le nombre de résultats est supérieur à 0, vous êtes averti via le groupe d’actions sélectionné.

> [!NOTE]
> Pour exécuter la règle d'alerte une fois par jour, sur tous les événements/journaux qui ont été créés le jour donné, remplacez la valeur de « Période » et de « Fréquence » par 1440, soit 24 heures.

#### <a name="alert-action-groups"></a>Groupes d’actions d’alerte

Utilisez un groupe d’actions pour spécifier un canal de notification. Pour voir les mécanismes de notification disponibles, sous **Groupes d’actions**, sélectionnez **Créer**.

![Mécanismes de notification disponibles dans la fenêtre « Ajouter un groupe d’actions »](media/backup-azure-monitoring-laworkspace/LA-AzureBackup-ActionGroup.png)

Vous pouvez satisfaire à toutes les exigences en matière d’alertes et de supervision à partir de Log Analytics seul, ou vous pouvez utiliser Log Analytics pour compléter les notifications intégrées.

Pour plus d’informations, consultez [Créer, afficher et gérer les alertes de journal avec Azure Monitor](../azure-monitor/platform/alerts-log.md) et [Créer et gérer des groupes d’actions dans le portail Azure](../azure-monitor/platform/action-groups.md).

### <a name="sample-kusto-queries"></a>Exemples de requêtes Kusto

Les graphiques par défaut vous fournissent des requêtes Kusto pour les scénarios de base sur lesquels vous pouvez créer des alertes. Vous pouvez également modifier les requêtes pour obtenir les données pour lesquelles vous voulez recevoir des alertes. Collez les exemples de requêtes Kusto suivants dans la page **Journaux**, puis créez des alertes sur les requêtes :

- Tous les travaux de sauvegarde réussis

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    ````

- Tous les travaux de sauvegarde ayant échoué

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Failed"
    ````

- Tous les travaux de sauvegarde de machines virtuelles Azure réussis

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="VM" and BackupManagementType=="IaaSVM"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Tous les travaux de sauvegarde de journal SQL réussis

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup" and JobOperationSubType=="Log"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="SQLDataBase" and BackupManagementType=="AzureWorkload"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Tous les travaux de l’agent Sauvegarde Azure réussis

    ````Kusto
    AddonAzureBackupJobs
    | where JobOperation=="Backup"
    | summarize arg_max(TimeGenerated,*) by JobUniqueId
    | where JobStatus=="Completed"
    | join kind=inner
    (
        CoreAzureBackup
        | where OperationName == "BackupItem"
        | where BackupItemType=="FileFolder" and BackupManagementType=="MAB"
        | distinct BackupItemUniqueId, BackupItemFriendlyName
    )
    on BackupItemUniqueId
    ````

- Stockage de sauvegarde consommé par élément de sauvegarde

    ````Kusto
    CoreAzureBackup
    //Get all Backup Items
    | where OperationName == "BackupItem"
    //Get distinct Backup Items
    | distinct BackupItemUniqueId, BackupItemFriendlyName
    | join kind=leftouter
    (AddonAzureBackupStorage
    | where OperationName == "StorageAssociation"
    //Get latest record for each Backup Item
    | summarize arg_max(TimeGenerated, *) by BackupItemUniqueId
    | project BackupItemUniqueId , StorageConsumedInMBs)
    on BackupItemUniqueId
    | project BackupItemUniqueId , BackupItemFriendlyName , StorageConsumedInMBs
    | sort by StorageConsumedInMBs desc
    ````

### <a name="diagnostic-data-update-frequency"></a>Fréquence de mise à jour des données de diagnostic

Les données de diagnostic provenant du coffre sont injectées dans l’espace de travail Log Analytics avec un certain décalage. Chaque événement arrive dans l’espace de travail Log Analytics *20 à 30 minutes* après son envoi (push) depuis le coffre Recovery Services. Voici plus d’informations sur le décalage :

- Dans toutes les solutions, les alertes intégrées du service de sauvegarde sont envoyées (push) dès qu’elles sont créées. Elles apparaissent donc généralement dans l’espace de travail Log Analytics après 20 à 30 minutes.
- Dans toutes les solutions, les travaux de sauvegarde et les travaux de restauration à la demande sont envoyés (push) dès qu’ils se *terminent*.
- Pour toutes les solutions, à l’exception de la sauvegarde SQL, les travaux de sauvegarde planifiés sont envoyés (push) dès qu’ils se *terminent*.
- Pour la sauvegarde SQL, comme des sauvegardes de journal peuvent se produire toutes les 15 minutes, les informations pour tous les travaux de sauvegarde planifiés terminés, y compris les journaux, sont traitées par lot et envoyées (push) toutes les 6 heures.
- Dans toutes les solutions, d’autres informations, comme l’élément de sauvegarde, la stratégie de sauvegarde, les points de récupération de sauvegarde, le stockage de sauvegarde, etc., sont envoyées (par push) au moins *une fois par jour*.
- Une modification dans la configuration de sauvegarde (comme la stratégie de changement ou de modification) déclenche un envoi (push) de toutes les informations de sauvegarde associées.

## <a name="using-the-recovery-services-vaults-activity-logs"></a>Utilisation des journaux d’activité du coffre Recovery Services

> [!CAUTION]
> Les étapes suivantes s’appliquent seulement aux *sauvegardes de machines virtuelles Azure*. Vous ne pouvez pas utiliser ces étapes pour des solutions comme l’agent Sauvegarde Azure, les sauvegardes SQL dans Azure ou Azure Files.

Vous pouvez aussi utiliser des journaux d’activité pour obtenir des notifications pour des événements comme la réussite des sauvegardes. Pour commencer, suivez ces étapes :

1. Connectez-vous au portail Azure.
2. Ouvrez le coffre Recovery Services approprié.
3. Dans les propriétés du coffre, ouvrez la section **Journal d’activité**.

Pour identifier le journal approprié et créer une alerte :

1. Vérifiez que vous recevez bien les journaux d’activité pour les sauvegardes réussies en appliquant les filtres montrés dans l’image suivante. Modifiez la valeur de **Intervalle de temps** selon les besoins pour afficher les enregistrements.

   ![Filtrage pour rechercher des journaux d’activité pour les sauvegardes de machines virtuelles Azure](media/backup-azure-monitoring-laworkspace/activitylogs-azurebackup-vmbackups.png)

2. Sélectionnez le nom de l’opération pour voir les détails correspondants.
3. Sélectionnez **Nouvelle règle d’alerte** pour ouvrir la page **Créer une règle**.
4. Créez une alerte en suivant les étapes de [Créer, afficher et gérer des alertes de journal d’activité avec Azure Monitor](../azure-monitor/platform/alerts-activity-log.md).

   ![Nouvelle règle d’alerte](media/backup-azure-monitoring-laworkspace/new-alert-rule.png)

Ici, la ressource est le coffre Recovery Services lui-même. Répétez les mêmes étapes pour tous les coffres pour lesquels vous voulez recevoir des notifications par le biais des journaux d’activité. La condition n’a pas de seuil, de période ou de fréquence, car cette alerte est basée sur des événements. Dès que le journal d’activité approprié est généré, l’alerte est déclenchée.

## <a name="using-log-analytics-to-monitor-at-scale"></a>Utilisation de Log Analytics pour superviser à grande échelle

Vous pouvez voir toutes les alertes créées à partir des journaux d’activité et des espaces de travail Log Analytics dans Azure Monitor. Ouvrez simplement le volet **Alertes** sur la gauche.

Même si vous pouvez recevoir des notifications via des journaux d’activité, nous vous recommandons vivement d’utiliser Log Analytics plutôt que des journaux d’activité pour la supervision à grande échelle. Voici pourquoi :

- **Scénarios limités** : Les notifications via des journaux d’activité s’appliquent seulement aux sauvegardes de machines virtuelles Azure. Les notifications doivent être configurées pour chaque coffre Recovery Services.
- **Ajustement de la définition** : L’activité de sauvegarde planifiée ne correspond pas à la définition la plus récente des journaux d’activité. Au lieu de cela, elle s’aligne sur les [journaux de ressources](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace). Cet alignement entraîne des effets inattendus quand les données qui transitent via le canal du journal d’activité changent.
- **Problèmes avec le canal du journal d’activité** : Dans les coffres Recovery Services, les journaux d’activité qui sont injectés depuis Sauvegarde Azure suivent un nouveau modèle. Malheureusement, ce changement affecte la génération des journaux d’activité dans Azure Government, Azure Allemagne et Azure Chine 21Vianet. Si les utilisateurs de ces services cloud créent ou configurent des alertes à partir de journaux d’activité dans Azure Monitor, les alertes ne sont pas déclenchées. De plus, dans toutes les régions publiques Azure, si un utilisateur [collecte des journaux d’activité Recovery Services dans un espace de travail Log Analytics](../azure-monitor/platform/activity-log.md), ces journaux n’apparaissent pas.

Utilisez un espace de travail Log Analytics pour la supervision et la génération d’alertes à grande échelle pour toutes vos charges de travail protégées par Sauvegarde Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour créer des requêtes personnalisées, consultez [Modèle de données Log Analytics](backup-azure-reports-data-model.md).
