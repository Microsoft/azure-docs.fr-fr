---
title: Utiliser des paramètres de diagnostic pour les coffres Recovery Services
description: Cet article décrit comment utiliser les anciens et nouveaux événements de diagnostic pour Sauvegarde Azure.
ms.topic: conceptual
ms.date: 10/30/2019
ms.openlocfilehash: 3d10053bae5148f33dba6d1207a81bdb16c37577
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002883"
---
# <a name="use-diagnostics-settings-for-recovery-services-vaults"></a>Utiliser des paramètres de diagnostic pour les coffres Recovery Services

Sauvegarde Azure envoie des événements de diagnostic qui peuvent être collectés et utilisés à des fins d’analyse, d’alerte et de création de rapports.

Vous pouvez configurer les paramètres de diagnostic d’un coffre Recovery Services via le portail Azure en accédant au coffre puis en cliquant sur **Paramètres de diagnostic**. L’option **+ Ajouter un paramètre de diagnostic** vous permet d’envoyer un ou plusieurs événements de diagnostic à un compte de stockage, un Event Hub ou un espace de travail Log Analytics.

![Volet Paramètres de diagnostic](./media/backup-azure-diagnostics-events/diagnostics-settings-blade.png)

## <a name="diagnostics-events-available-for-azure-backup-users"></a>Événements de diagnostic disponibles pour les utilisateurs de Sauvegarde Azure

Sauvegarde Azure fournit les événements de diagnostic suivant. Chaque événement fournit des données détaillées sur un ensemble spécifique d’artefacts liés à la sauvegarde :

* CoreAzureBackup
* AddonAzureBackupAlerts
* AddonAzureBackupProtectedInstance
* AddonAzureBackupJobs
* AddonAzureBackupPolicy
* AddonAzureBackupStorage

Si vous utilisez toujours l’[événement hérité](#legacy-event) AzureBackupReport, nous vous recommandons de passer à l’utilisation des événements ci-dessus.

Pour plus d’informations, consultez [Modèle de données pour les événements de diagnostics de Sauvegarde Azure](./backup-azure-reports-data-model.md).

Les données de ces événements peuvent être envoyées à un compte de stockage, à un espace de travail Log Analytics ou à un Event Hub. Si vous envoyez ces données à un espace de travail Log Analytics, sélectionnez le bouton bascule **Spécifique de la** sur l’écran **Paramètres de diagnostic**. Pour plus d’informations, consultez les sections suivantes.

## <a name="use-diagnostics-settings-with-log-analytics"></a>Utiliser des paramètres de diagnostic avec Log Analytics

Vous pouvez maintenant utiliser Sauvegarde Azure pour envoyer des données de diagnostic de coffre à des tables Log Analytics dédiées à la sauvegarde. Ces tables sont appelées [tables spécifiques de la ressource](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).

Pour envoyer les données de diagnostic de votre coffre à Log Analytics :

1. Accédez à votre coffre, puis sélectionnez **Paramètres de diagnostic**. Sélectionnez **+ Ajouter un paramètre de diagnostic**.
1. Donnez un nom au paramètre de diagnostic.
1. Cochez la case **Envoyer à Log Analytics**, puis sélectionnez un espace de travail Log Analytics.
1. Sélectionnez le bouton bascule **Spécifique de la ressource**, puis choisissez les six événements suivants : **CoreAzureBackup**, **AddonAzureBackupJobs**, **AddonAzureBackupAlerts**, **AddonAzureBackupPolicy**, **AddonAzureBackupStorage** et **AddonAzureBackupProtectedInstance**.
1. Sélectionnez **Enregistrer**.

   ![Mode Spécifique de la ressource](./media/backup-azure-diagnostics-events/resource-specific-blade.png)

Une fois que les données circulent dans l’espace de travail Log Analytics, les tables dédiées pour chacun de ces événements sont créées dans votre espace de travail. Vous pouvez interroger directement n’importe laquelle de ces tables. Vous pouvez également effectuer des jointures ou des unions entre ces tables si nécessaire.

> [!IMPORTANT]
> Les six événements, à savoir CoreAzureBackup, AddonAzureBackupJobs, AddonAzureBackupAlerts, AddonAzureBackupPolicy, AddonAzureBackupStorage et AddonAzureBackupProtectedInstance sont pris en charge *uniquement* dans le mode Spécifique de la ressource dans [Rapports de sauvegarde](./configure-reports.md). *Si vous essayez d’envoyer des données pour ces six événements en mode Diagnostics Azure, aucune donnée n’est visible dans Rapports de sauvegarde.*

## <a name="legacy-event"></a>Événement hérité

Traditionnellement, toutes les données de diagnostics relatives à la sauvegarde d’un coffre figuraient dans un seul événement appelé « AzureBackupReport ». Les six événements décrits ici sont, par essence, une décomposition de toutes les données contenues dans AzureBackupReport.

Actuellement, nous continuons à prendre en charge l’événement AzureBackupReport pour la compatibilité descendante, dans les cas où les utilisateurs ont des requêtes personnalisées existantes sur cet événement, par exemple, des alertes de journal personnalisées ou des visualisations personnalisées. *Nous vous recommandons de passer aux [derniers événements](#diagnostics-events-available-for-azure-backup-users) dès que possible*. Les nouveaux événements :

* Facilitent considérablement l’utilisation des données dans les requêtes de journal.
* Améliorent la détectabilité des schémas et leur structure.
* Améliorent les performances au niveau de la latence d’ingestion et des délais de requêtes.

*L’événement hérité en mode Diagnostics Azure sera finalement déconseillé. Choisir les nouveaux événements peut vous aider à éviter des migrations complexes à une date ultérieure*. Notre [solution de création de rapports](./configure-reports.md) qui utilise Log Analytics cessera également de prendre en charge les données de l’événement hérité.

### <a name="steps-to-move-to-new-diagnostics-settings-for-a-log-analytics-workspace"></a>Étapes à suivre pour passer aux nouveaux paramètres de diagnostic pour un espace de travail Log Analytics

1. Identifiez les coffres qui envoient des données aux espaces de travail Log Analytics à l’aide de l’événement hérité et les abonnements auxquels ils appartiennent. Cet article décrit comment utiliser les anciens et nouveaux événements de diagnostic pour Sauvegarde Azure.

    ````Kusto
    let RangeStart = startofday(ago(3d));
    let VaultUnderAzureDiagnostics = (){
        AzureDiagnostics
        | where TimeGenerated >= RangeStart | where Category == "AzureBackupReport" and OperationName == "Vault" and SchemaVersion_s == "V2"
        | summarize arg_max(TimeGenerated, *) by ResourceId
        | project ResourceId, Category};
    let VaultUnderResourceSpecific = (){
        CoreAzureBackup
        | where TimeGenerated >= RangeStart | where OperationName == "Vault"
        | summarize arg_max(TimeGenerated, *) by ResourceId
        | project ResourceId, Category};
        // Some Workspaces will not have AzureDiagnostics Table, so you need to use isFuzzy
    let CombinedVaultTable = (){
        union isfuzzy = true
        (VaultUnderAzureDiagnostics() ),
        (VaultUnderResourceSpecific() )
        | distinct ResourceId, Category};
    CombinedVaultTable | where Category == "AzureBackupReport"
    | join kind = leftanti (
    CombinedVaultTable | where Category == "CoreAzureBackup"
    ) on ResourceId
    | parse ResourceId with * "SUBSCRIPTIONS/" SubscriptionId:string "/RESOURCEGROUPS" * "MICROSOFT.RECOVERYSERVICES/VAULTS/" VaultName:string
    | project ResourceId, SubscriptionId, VaultName
    ````

    Vous trouverez ci-dessous une capture d’écran de la requête en cours d’exécution dans l’un des espaces de travail :

    ![Requête d’espace de travail](./media/backup-azure-diagnostics-events/workspace-query.png)

2. Utilisez les [définitions Azure Policy intégrées](./azure-policy-configure-diagnostics.md) à Sauvegarde Azure pour ajouter un nouveau paramètre de diagnostic pour tous les coffres au sein d’une étendue spécifiée. Cette stratégie ajoute un paramètre de diagnostic aux coffres qui n’ont pas de paramètre de diagnostic ou qui ne disposent que d’un paramètre de diagnostic hérité. Cette stratégie peut être attribuée à un abonnement ou groupe de ressources entier à la fois. Vous aurez besoin d’un accès Propriétaire à chaque abonnement pour lequel la stratégie est attribuée.

Vous pouvez choisir d’avoir des paramètres de diagnostic distincts pour AzureBackupReport et les six nouveaux événements jusqu’à ce que vous ayez migré toutes vos requêtes personnalisées pour utiliser les données des nouvelles tables. L’image suivante montre un exemple de coffre avec deux paramètres de diagnostic. Le premier paramètre, nommé **Setting1** envoie des données d’événement AzureBackupReport à un espace de travail Log Analytics dans le mode Diagnostics Azure. Le deuxième paramètre, nommé **Setting2** envoie les données des six nouveaux événements Sauvegarde Azure à un espace de travail Log Analytics dans le mode Spécifique de la ressource.

![Deux paramètres](./media/backup-azure-diagnostics-events/two-settings-example.png)

> [!IMPORTANT]
> L’événement AzureBackupReport est pris en charge *uniquement* dans le mode Diagnostics Azure. *Si vous essayez d’envoyer des données pour cet événement en mode Spécifique de la ressource, aucune donnée n’est dirigée vers l’espace de travail Log Analytics.*

> [!NOTE]
> Le bouton bascule **Diagnostics Azure** ou **Spécifique de la ressource** n’apparaît que si l’utilisateur sélectionne **Envoyer à Log Analytics**. Pour envoyer des données à un compte de stockage ou à un Event Hub, un utilisateur sélectionne la destination requise et coche les événements souhaités, sans aucune entrée supplémentaire. Là encore, nous vous recommandons de ne plus choisir l’événement hérité AzureBackupReport.

## <a name="send-azure-site-recovery-events-to-log-analytics"></a>Envoyer des événements d’Azure Site Recovery à Log Analytics

Les événements Sauvegarde Azure et Azure Site Recovery sont envoyés à partir du même coffre Recovery Services. Azure Site Recovery n’est actuellement pas disponible pour les tables spécifiques de la ressource. Les utilisateurs qui souhaitent envoyer des événements Azure Site Recovery à Log Analytics sont redirigés pour utiliser le mode Diagnostics Azure *uniquement*, comme indiqué dans l’image. *Choisir le mode Spécifique de la ressource pour les événements Azure Site Recovery empêchera l’envoi des données requises à l’espace de travail Log Analytics*.

![Événements Site Recovery](./media/backup-azure-diagnostics-events/site-recovery-settings.png)

Pour résumer :

* Si vous avez déjà configuré Diagnostics Log Analytics avec Diagnostics Azure et que vous avez écrit des requêtes personnalisées, *ne modifiez pas* ce paramètre jusqu’à ce que vous migriez vos requêtes pour utiliser les données des nouveaux événements.
* Si vous souhaitez également effectuer une intégration dans de nouvelles tables, comme cela est recommandé, créez un **nouveau** paramètre de diagnostics, choisissez **Spécifique de la ressource**, puis sélectionnez les six nouveaux événements.
* Si vous envoyez actuellement des événements Azure Site Recovery à Log Analytics, *ne choisissez pas* le mode Spécifique de la ressource pour ces événements. Sinon, les données de ces événements ne seront pas envoyées à votre espace de travail Log Analytics. Au lieu de cela, créez un paramètre de diagnostic supplémentaire, sélectionnez **Diagnostics Azure**, puis choisissez les événements Azure Site Recovery pertinents.

L’image suivante représente l’exemple d’un utilisateur ayant trois paramètres de diagnostic pour un coffre. Le premier paramètre, nommé **Setting1** envoie des données d’événement AzureBackupReport à un espace de travail Log Analytics dans le mode Diagnostics Azure. Le deuxième paramètre, nommé **Setting2** envoie les données des six nouveaux événements Sauvegarde Azure à un espace de travail Log Analytics dans le mode Spécifique de la ressource. Le troisième paramètre, nommé **Setting3** envoie des données d’événement Azure Site Recovery à un espace de travail Log Analytics dans le mode Diagnostics Azure.

![Trois paramètres](./media/backup-azure-diagnostics-events/three-settings-example.png)

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur le modèle de données Log Analytics pour les événements de diagnostic](./backup-azure-reports-data-model.md)
