---
title: Supervision du Stockage Blob Azure | Microsoft Docs
description: Découvrez comment superviser les performances et la disponibilité du Stockage Blob Azure. Supervisez les données du Stockage Blob Azure, apprenez-en davantage sur la configuration, et analysez les données de métriques et de journal.
author: normesta
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 10/26/2020
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 9224d02e36dbca96d3e54946330d3135ff811829
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590764"
---
# <a name="monitoring-azure-blob-storage"></a>Supervision du Stockage Blob Azure

Lorsque vous avez des applications critiques et des processus métier basés sur des ressources Azure, vous voulez superviser ces ressources pour connaître leur disponibilité, leurs performances et leur fonctionnement. Cet article décrit les données de supervision générées par le Stockage Blob Azure et comment vous pouvez utiliser les fonctionnalités d’Azure Monitor pour analyser les alertes sur ces données.

> [!NOTE]
> Les journaux de stockage Azure dans Azure Monitor sont en préversion publique et sont disponibles pour le test en préversion dans toutes les régions de cloud public. Cette préversion active les journaux des objets blob (qui incluent Azure Data Lake Storage Gen2), les fichiers, les files d’attente et les tables. Cette fonctionnalité est disponible pour tous les comptes de stockage créés avec le modèle de déploiement Resource Manager. Voir [Vue d’ensemble des comptes de stockage](../common/storage-account-overview.md).

## <a name="monitor-overview"></a>Présentation de Monitor

La page **Vue d’ensemble** du portail Azure pour chaque ressource du Stockage Blob comprend un bref aperçu de l’utilisation de la ressource, comme les demandes et la facturation horaire. Ces informations sont utiles, mais elles ne constituent qu’une petite quantité des données de supervision disponibles. Certaines de ces données sont collectées automatiquement et peuvent être analysées dès que vous créez la ressource. Vous pouvez activer d’autres types de collecte de données avec une certaine configuration.

## <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?
Le Stockage Blob Azure crée des données de supervision à l’aide d’[Azure Monitor](../../azure-monitor/overview.md), qui est un service de supervision de pile complète dans Azure. Azure Monitor fournit un ensemble complet de fonctionnalités pour superviser vos ressources Azure, ainsi que les ressources locales et celles présentes dans d’autres clouds. 

Commencez avec l’article [Supervision de ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md) qui décrit les éléments suivants :

- Qu’est-ce qu’Azure Monitor ?
- Coûts associés à la supervision
- Analyse des données collectées dans Azure
- Configuration de la collecte des données
- Outils standard dans Azure pour l’analyse et la génération d’alertes sur les données analysées

Les sections suivantes s’appuient sur cet article en décrivant les données spécifiques collectées à partir du Stockage Azure. Les exemples montrent comment configurer la collecte de données et analyser ces données avec les outils Azure.

## <a name="monitoring-data"></a>Données de surveillance

Le Stockage Blob Azure collecte les mêmes types de données de supervision que d’autres ressources Azure, lesquelles sont décrites dans [Supervision des données de ressources Azure](../../azure-monitor/insights/monitor-azure-resource.md#monitoring-data). 

Pour obtenir des informations détaillées sur les métriques et les métriques de journaux créées par le service Stockage Blob Azure, consultez [Informations de référence sur les données de supervision du service Stockage Blob Azure](monitor-blob-storage-reference.md).

Les métriques et les journaux d’Azure Monitor ne prennent en charge que les comptes de stockage Azure Resource Manager. Azure Monitor ne prend pas en charge les comptes de stockage classiques. Si vous souhaitez utiliser des métriques ou des journaux sur un compte de stockage classique, vous devez migrer vers un compte de stockage Azure Resource Manager. Voir [Migrer vers Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

Vous pouvez continuer à utiliser les métriques et les journaux classiques si vous le souhaitez. En fait, les métriques et les journaux classiques sont disponibles parallèlement aux métriques et journaux dans Azure Monitor. La prise en charge reste en place jusqu’à ce que le stockage Azure mette fin au service sur les métriques et les journaux hérités.

## <a name="collection-and-routing"></a>Collecte et routage

Les métriques de plateforme et le journal d'activité sont collectés automatiquement, mais ils peuvent être acheminés vers d'autres emplacements à l'aide d'un paramètre de diagnostic. 

Pour collecter des journaux de ressources, vous devez créer un paramètre de diagnostic. Lorsque vous créez le paramètre, choisissez **Blob** comme type de stockage pour lequel vous souhaitez activer les journaux. Ensuite, spécifiez l’une des catégories d’opérations suivantes pour laquelle vous souhaitez collecter les journaux. 

| Category | Description |
|:---|:---|
| StorageRead | Opérations de lecture sur des objets. |
| StorageWrite | Opérations d’écriture sur des objets. |
| StorageDelete | Opérations de suppression sur des objets. |

> [!NOTE]
> Data Lake Storage Gen2 n’apparaît pas en tant que type de stockage. Cela est dû au fait que Data Lake Storage Gen2 est un ensemble de fonctionnalités disponibles pour le stockage Blob. 

## <a name="creating-a-diagnostic-setting"></a>Création d’un paramètre de diagnostic

Vous pouvez créer un paramètre de diagnostic en utilisant le portail Azure, PowerShell, l’interface de ligne de commande Azure ou un modèle Azure Resource Manager. 

Pour obtenir des instructions générales, consultez [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](../../azure-monitor/platform/diagnostic-settings.md).

> [!NOTE]
> Les journaux de stockage Azure dans Azure Monitor sont en préversion publique et sont disponibles pour le test en préversion dans toutes les régions de cloud public. Cette préversion active les journaux des objets blob (qui incluent Azure Data Lake Storage Gen2), les fichiers, les files d’attente et les tables. Cette fonctionnalité est disponible pour tous les comptes de stockage créés avec le modèle de déploiement Resource Manager. Voir [Vue d’ensemble des comptes de stockage](../common/storage-account-overview.md).

### <a name="azure-portal"></a>[Azure portal](#tab/azure-portal)

1. Connectez-vous au portail Azure.

2. Accédez à votre compte de stockage.

3. Dans la section **Supervision**, cliquez sur **Paramètres de diagnostic (préversion)** .

   > [!div class="mx-imgBorder"]
   > ![Portail - Journaux de diagnostics](media/monitor-blob-storage/diagnostic-logs-settings-pane.png)   

4. Choisissez **Blob** comme type de stockage pour lequel vous souhaitez activer les journaux.

5. Cliquez sur **Ajouter le paramètre de diagnostic**.

   > [!div class="mx-imgBorder"]
   > ![portail – Journaux de ressources – ajouter un paramètre de diagnostic](media/monitor-blob-storage/diagnostic-logs-settings-pane-2.png)

   La page **Paramètres de diagnostic** s’affiche.

   > [!div class="mx-imgBorder"]
   > ![Page Journaux de ressources](media/monitor-blob-storage/diagnostic-logs-page.png)

6. Dans le champ **Nom** de la page, entrez un nom pour ce paramètre de journal des ressources. Sélectionnez ensuite les opérations que vous souhaitez consigner (opérations de lecture, d’écriture et de suppression), ainsi que l’emplacement où vous souhaitez que les journaux soient envoyés.

#### <a name="archive-logs-to-a-storage-account"></a>Archiver les journaux dans un compte de stockage

Si vous choisissez d’archiver vos journaux dans un compte de stockage, vous paierez pour le volume des journaux qui sont envoyés au compte de stockage. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Cochez la case **Archiver dans un compte de stockage**, puis cliquez sur le bouton **Configurer**.

   > [!div class="mx-imgBorder"]   
   > ![Page Paramètres de diagnostic – Stockage d’archive](media/monitor-blob-storage/diagnostic-logs-settings-pane-archive-storage.png)

2. Dans la liste déroulante **Compte de stockage**, sélectionnez le compte de stockage dans lequel vous souhaitez archiver vos journaux, cliquez sur le bouton **OK**, puis cliquez sur le bouton **Enregistrer**.

   > [!NOTE]
   > Avant de choisir un compte de stockage comme destination d’exportation, consultez [Archiver les journaux de ressources Azure](../../azure-monitor/platform/resource-logs.md#send-to-azure-storage) pour comprendre les conditions préalables relatives au compte de stockage.

#### <a name="stream-logs-to-azure-event-hubs"></a>Diffuser les journaux en continu vers Azure Event Hubs

Si vous choisissez de diffuser vos journaux vers un Event Hub, vous paierez pour le volume des journaux envoyés à l’Event Hub. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Cochez la case **Diffuser sur un Event Hub**, puis cliquez sur le bouton **Configurer**.

2. Dans le volet **Sélectionner un Event Hub**, choisissez l’espace de noms, le nom et le nom de la stratégie de l’Event Hub vers lequel vous souhaitez diffuser vos journaux en continu. 

   > [!div class="mx-imgBorder"]
   > ![Page Paramètres de diagnostic – Event Hub](media/monitor-blob-storage/diagnostic-logs-settings-pane-event-hub.png)

3. Cliquez sur le bouton **OK**, puis cliquez sur le bouton **Enregistrer**.

#### <a name="send-logs-to-azure-log-analytics"></a>Envoyer des journaux à Azure Log Analytics

1. Cochez la case **Envoyer à Log Analytics**, sélectionnez un espace de travail Log Analytics, puis cliquez sur le bouton **Enregistrer**.

   > [!div class="mx-imgBorder"]   
   > ![Page Paramètres de diagnostic – Log Analytics](media/monitor-blob-storage/diagnostic-logs-settings-pane-log-analytics.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Ouvrez une fenêtre de commande Windows PowerShell, puis connectez-vous à votre abonnement Azure à l’aide de la commande `Connect-AzAccount`. Ensuite, suivez les instructions à l’écran.

   ```powershell
   Connect-AzAccount
   ```

2. Définissez votre abonnement actif sur abonnement du compte de stockage pour lequel vous souhaitez activer la journalisation.

   ```powershell
   Set-AzContext -SubscriptionId <subscription-id>
   ```

#### <a name="archive-logs-to-a-storage-account"></a>Archiver les journaux dans un compte de stockage

Si vous choisissez d’archiver vos journaux dans un compte de stockage, vous paierez pour le volume des journaux qui sont envoyés au compte de stockage. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Activez les journaux à l’aide de la cmdlet PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) avec le paramètre `StorageAccountId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -StorageAccountId <storage-account-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Remplacez l’espace réservé `<storage-service-resource--id>` de cet extrait par l’ID de ressource du service Blob. Vous pouvez trouver l’ID de ressource dans le portail Azure en ouvrant la page **Propriétés** de votre compte de stockage.

Vous pouvez utiliser `StorageRead`, `StorageWrite` et `StorageDelete` comme valeur du paramètre **Category**.

Voici un exemple :

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/blobServices/default -StorageAccountId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount -Enabled $true -Category StorageWrite,StorageDelete`

Pour obtenir une description de chaque paramètre, consultez [Archiver des journaux de ressources Azure via Azure PowerShell](../../azure-monitor/platform/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmettre en continu des journaux d’activité vers un hub d’événements

Si vous choisissez de diffuser vos journaux vers un Event Hub, vous paierez pour le volume des journaux envoyés à l’Event Hub. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Activez les journaux à l’aide de la cmdlet PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) avec le paramètre `EventHubAuthorizationRuleId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -EventHubAuthorizationRuleId <event-hub-namespace-and-key-name> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Voici un exemple :

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/blobServices/default -EventHubAuthorizationRuleId /subscriptions/20884142-a14v3-4234-5450-08b10c09f4/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey -Enabled $true -Category StorageDelete`

Pour obtenir une description de chaque paramètre, consultez [Diffuser des données en continu vers Event Hubs via des cmdlets PowerShell](../../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envoyer des journaux d’activité à Log Analytics

Activez les journaux à l’aide de la cmdlet PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) avec le paramètre `WorkspaceId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -WorkspaceId <log-analytics-workspace-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Voici un exemple :

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/blobServices/default -WorkspaceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace -Enabled $true -Category StorageDelete`

Pour plus d’informations, consultez [Diffuser des journaux de ressources Azure en continu vers l’espace de travail Log Analytics dans Azure Monitor](../../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Commencez par ouvrir [Azure Cloud Shell](../../cloud-shell/overview.md) ou, si vous avez [installé](/cli/azure/install-azure-cli) Azure CLI localement, ouvrez une application console de commandes telle que Windows PowerShell.

2. Si votre identité est associée à plusieurs abonnements, définissez l’abonnement du compte de stockage pour lequel vous souhaitez activer les journaux comme abonnement actif.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

#### <a name="archive-logs-to-a-storage-account"></a>Archiver les journaux dans un compte de stockage

Si vous choisissez d’archiver vos journaux dans un compte de stockage, vous paierez pour le volume des journaux qui sont envoyés au compte de stockage. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Activez les journaux à l’aide de la commande [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --storage-account <storage-account-name> --resource <storage-service-resource-id> --resource-group <resource-group> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Remplacez l’espace réservé `<storage-service-resource--id>` dans cet extrait de code par le service Stockage Blob de l’ID de ressource. Vous pouvez trouver l’ID de ressource dans le portail Azure en ouvrant la page **Propriétés** de votre compte de stockage.

Vous pouvez utiliser `StorageRead`, `StorageWrite` et `StorageDelete` comme valeur du paramètre **Category**.

Voici un exemple :

`az monitor diagnostic-settings create --name setting1 --storage-account mystorageaccount --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/blobServices/default --resource-group myresourcegroup --logs '[{"category": StorageWrite, "enabled": true, "retentionPolicy": {"days": 90, "enabled": true}}]'`

Pour obtenir une description de chaque paramètre, consultez [Archiver des journaux de ressources via l’interface de ligne de commande Azure](../../azure-monitor/platform/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmettre en continu des journaux d’activité vers un hub d’événements

Si vous choisissez de diffuser vos journaux vers un Event Hub, vous paierez pour le volume des journaux envoyés à l’Event Hub. Pour connaître les tarifs spécifiques, consultez la section **Journaux de la plateforme** de la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Activez les journaux à l’aide de la commande [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --event-hub <event-hub-name> --event-hub-rule <event-hub-namespace-and-key-name> --resource <storage-account-resource-id> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Voici un exemple :

`az monitor diagnostic-settings create --name setting1 --event-hub myeventhub --event-hub-rule /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/blobServices/default --logs '[{"category": StorageDelete, "enabled": true }]'`

Pour obtenir une description de chaque paramètre, consultez [Diffuser des données en continu vers Event Hubs via l’interface de ligne de commande Azure](../../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envoyer des journaux d’activité à Log Analytics

Activez les journaux à l’aide de la commande [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --workspace <log-analytics-workspace-resource-id> --resource <storage-account-resource-id> --logs '[{"category": <category name>, "enabled": true "retentionPolicy": {"days": <days>, "enabled": <retention-bool}}]'
```

Voici un exemple :

`az monitor diagnostic-settings create --name setting1 --workspace /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/blobServices/default --logs '[{"category": StorageDelete, "enabled": true ]'`

 Pour plus d’informations, consultez [Diffuser des journaux de ressources Azure en continu vers l’espace de travail Log Analytics dans Azure Monitor](../../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).

### <a name="template"></a>[Modèle](#tab/template)

Pour afficher un modèle Azure Resource Manager qui crée un paramètre de diagnostic, consultez [Paramètre de diagnostic pour Stockage Azure](../../azure-monitor/samples/resource-manager-diagnostic-settings.md#diagnostic-setting-for-azure-storage).

---

## <a name="analyzing-metrics"></a>Analyse des métriques

Vous pouvez analyser les métriques de Stockage Azure avec des métriques issues d’autres services Azure à l’aide de Metrics Explorer. Ouvrez Metrics Explorer en choisissant **Métriques** dans le menu **Azure Monitor**. Pour plus d’informations sur l’utilisation de cet outil, consultez [Prise en main d’Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md). 

Cet exemple montre comment afficher les **Transactions** au niveau du compte.

![Capture d’écran d’accès aux mesures dans le Portail Azure](./media/monitor-blob-storage/access-metrics-portal.png)

Pour les métriques prenant en charge des dimensions, vous pouvez les filtrer avec la valeur de dimension souhaitée. Cet exemple montre comment afficher les **Transactions** au niveau du compte sur une opération spécifique en sélectionnant des valeurs pour la dimension **Nom de l’API**.

![Capture d’écran d’accès aux mesures avec une dimension dans le Portail Azure](./media/monitor-blob-storage/access-metrics-portal-with-dimension.png)

Pour obtenir la liste complète des dimensions prises en charge par le stockage Azure, consultez [Dimensions des métriques](monitor-blob-storage-reference.md#metrics-dimensions).

Les métriques du service Stockage Blob Azure se trouvent dans les espaces de noms suivants : 

- Microsoft.Storage/storageAccounts
- Microsoft.Storage/storageAccounts/blobServices

Pour obtenir la liste de toutes les métriques de prise en charge d’Azure Monitor, ce qui inclut le Stockage Blob Azure, consultez [Métriques de prise en charge avec Azure Monitor](../../azure-monitor/platform/metrics-supported.md).


### <a name="accessing-metrics"></a>Accès aux métriques

> [!TIP]
> Pour afficher des exemples Azure CLI ou .NET, choisissez l’onglet correspondant listé ici.

### <a name="net"></a>[.NET](#tab/azure-portal)

Azure Monitor fournit des [kits de développement logiciel (SDK) .NET](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/) pour lire des définitions et valeurs de mesures. L’[exemple de code](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) montre comment utiliser le kit de développement logiciel (SDK) avec des paramètres différents. Vous devez utiliser `0.18.0-preview` ou une version ultérieure pour les mesures de stockage.
 
Dans ces exemples, remplacez l’espace réservé `<resource-ID>` par l’ID de ressource du compte de stockage complet ou du service Stockage Blob. Vous pouvez trouver les ID de ces ressources sur les pages **Propriétés** de votre compte de stockage sur le Portail Azure.

Remplacez la variable `<subscription-ID>` par l’ID de votre abonnement. Pour obtenir des conseils sur la façon d’obtenir des valeurs pour `<tenant-ID>`, `<application-ID>` et `<AccessKey>`, consultez [Utiliser le portail pour créer une application et un principal du service Azure AD pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md). 

#### <a name="list-the-account-level-metric-definition"></a>Dresser la liste de la définition des métriques de niveau compte

L’exemple suivant montre comment répertorier une définition de mesures au niveau du compte :

```csharp
    public static async Task ListStorageMetricDefinition()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";


        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;
        IEnumerable<MetricDefinition> metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceId, cancellationToken: new CancellationToken());

        foreach (var metricDefinition in metricDefinitions)
        {
            // Enumrate metric definition:
            //    Id
            //    ResourceId
            //    Name
            //    Unit
            //    MetricAvailabilities
            //    PrimaryAggregationType
            //    Dimensions
            //    IsDimensionRequired
        }
    }

```

#### <a name="reading-account-level-metric-values"></a>Lecture des valeurs des métriques de niveau compte

L’exemple suivant montre comment lire des données `UsedCapacity` au niveau du compte :

```csharp
    public static async Task ReadStorageMetricValue()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;

        Response = await readOnlyClient.Metrics.ListAsync(
            resourceUri: resourceId,
            timespan: timeSpan,
            interval: System.TimeSpan.FromHours(1),
            metricnames: "UsedCapacity",

            aggregation: "Average",
            resultType: ResultType.Data,
            cancellationToken: CancellationToken.None);

        foreach (var metric in Response.Value)
        {
            // Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

#### <a name="reading-multidimensional-metric-values"></a>Lecture des valeurs des métriques multidimensionnelles

Pour les mesures multidimensionnelles, vous devez définir le filtre des métadonnées si vous souhaitez lire les données métriques sur des valeurs de dimension spécifiques.

L’exemple suivant montre comment lire les données de mesures sur la métrique prenant en charge plusieurs dimensions :

```csharp
    public static async Task ReadStorageMetricValueTest()
    {
        // Resource ID for blob storage
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/blobServices/default";
        var subscriptionId = "<subscription-ID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;
        // It's applicable to define meta data filter when a metric support dimension
        // More conditions can be added with the 'or' and 'and' operators, example: BlobType eq 'BlockBlob' or BlobType eq 'PageBlob'
        ODataQuery<MetadataValue> odataFilterMetrics = new ODataQuery<MetadataValue>(
            string.Format("BlobType eq '{0}'", "BlockBlob"));

        Response = readOnlyClient.Metrics.List(
                        resourceUri: resourceId,
                        timespan: timeSpan,
                        interval: System.TimeSpan.FromHours(1),
                        metricnames: "BlobCapacity",
                        odataQuery: odataFilterMetrics,
                        aggregation: "Average",
                        resultType: ResultType.Data);

        foreach (var metric in Response.Value)
        {
            //Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

#### <a name="list-the-metric-definition"></a>Dresser la liste de la définition des métriques

Vous pouvez afficher la définition des métriques de votre compte de stockage ou du service de Stockage Blob. Utilisez l’applet de commande [Get-AzMetricDefinition](/powershell/module/az.monitor/get-azmetricdefinition).

Dans cet exemple, remplacez l’espace réservé `<resource-ID>` par l’ID de ressource du compte de stockage complet ou par l’ID de ressource du service Stockage Blob.  Vous pouvez trouver les ID de ces ressources sur les pages **Propriétés** de votre compte de stockage sur le Portail Azure.

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetricDefinition -ResourceId $resourceId
```

#### <a name="reading-metric-values"></a>Lecture des valeurs des métriques

Vous pouvez lire les valeurs des métriques de niveau compte de votre compte de stockage ou du service Stockage Blob. Utilisez l’applet de commande [Get-AzMetric](/powershell/module/Az.Monitor/Get-AzMetric).

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetric -ResourceId $resourceId -MetricNames "UsedCapacity" -TimeGrain 01:00:00
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

#### <a name="list-the-account-level-metric-definition"></a>Dresser la liste de la définition des métriques de niveau compte

Vous pouvez afficher la définition des métriques de votre compte de stockage ou du service de Stockage Blob. Utilisez la commande [az monitor metrics list-definitions](/cli/azure/monitor/metrics#az-monitor-metrics-list-definitions).
 
Dans cet exemple, remplacez l’espace réservé `<resource-ID>` par l’ID de ressource du compte de stockage complet ou par l’ID de ressource du service Stockage Blob. Vous pouvez trouver les ID de ces ressources sur les pages **Propriétés** de votre compte de stockage sur le Portail Azure.

```azurecli-interactive
   az monitor metrics list-definitions --resource <resource-ID>
```

#### <a name="read-account-level-metric-values"></a>Lire les valeurs des métriques de niveau compte

Vous pouvez lire les valeurs des métriques de votre compte de stockage ou du service Stockage Blob. Utilisez la commande [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list).

```azurecli-interactive
   az monitor metrics list --resource <resource-ID> --metric "UsedCapacity" --interval PT1H
```
### <a name="template"></a>[Modèle](#tab/template)

N/A.

---

## <a name="analyzing-logs"></a>Analyse des journaux d’activité

Vous pouvez accéder aux journaux des ressources en tant qu’objet blob dans un compte de stockage, en tant que données d’événement ou par le biais de requêtes Log Analytics.

Pour obtenir des informations de référence détaillées sur les champs qui apparaissent dans ces journaux, consultez [Informations de référence sur les données de supervision du Stockage Blob Azure](monitor-blob-storage-reference.md).

> [!NOTE]
> Les journaux de stockage Azure dans Azure Monitor sont en préversion publique et sont disponibles pour le test en préversion dans toutes les régions de cloud public. Cette préversion active les journaux pour les objets blob (qui comprennent Azure Data Lake Storage Gen2), les fichiers, les files d’attente, les tables, les comptes de stockage Premium dans les comptes de stockage à usage général v1 et à usage général v2. Les comptes de stockage classiques ne sont pas pris en charge.

Les entrées de journal sont créées uniquement si des demandes sont effectuées sur le point de terminaison de service. Par exemple, si un compte de stockage a une activité dans son point de terminaison de blob, mais pas dans ses points de terminaison de table ou de file d’attente, seuls les journaux d’activité qui appartiennent au service Blob sont créés. Les journaux de stockage Azure contiennent des informations détaillées sur les demandes ayant réussi ou échoué pour un service de stockage. Ces informations peuvent servir à analyser des demandes individuelles et à diagnostiquer les problèmes au niveau d'un service de stockage. Les demandes sont enregistrées sur la base du meilleur effort.

### <a name="log-authenticated-requests"></a>Journaliser des requêtes authentifiées

 Les types de demandes authentifiées suivants sont enregistrés :

- Demandes ayant réussi
- Demandes ayant échoué, y compris les erreurs de délai d’expiration, limitation, réseau, autorisation et autres erreurs
- Demandes utilisant une signature d’accès partagé (SAS) ou OAuth, y compris les demandes ayant réussi et ayant échoué
- Demandes de données d’analyse (données de journal classique dans le conteneur **$logs** et données de métriques de classe dans les tables **$metric**)

Les demandes effectuées par le service Stockage Blob lui-même, comme la création ou la suppression d’un journal, ne sont pas journalisées. Pour obtenir une liste complète des données enregistrées, consultez [Opérations et messages d’état enregistrés de stockage](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) et [Format de journal de stockage](monitor-blob-storage-reference.md).

### <a name="log-anonymous-requests"></a>Journaliser des requêtes anonymes

 Les types de demandes anonymes suivants sont enregistrés :

- Demandes ayant réussi
- Erreurs de serveur
- Erreurs d’expiration pour le client et le serveur
- Demandes GET ayant échoué avec le code d’erreur 304 (non modifié)

Aucune autre demande anonyme ayant échoué n'est enregistrée. Pour obtenir une liste complète des données enregistrées, consultez [Opérations et messages d’état enregistrés de stockage](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) et [Format de journal de stockage](monitor-blob-storage-reference.md).

### <a name="accessing-logs-in-a-storage-account"></a>Accès aux journaux dans un compte de stockage

Les journaux apparaissent sous forme d’objets blob stockés dans un conteneur du compte de stockage cible. Les données sont collectées et stockées dans un objet blob individuel sous la forme d’une charge utile JSON délimitée par une ligne. Le nom d’objet blob suit cette convention d’affectation de noms :

`https://<destination-storage-account>.blob.core.windows.net/insights-logs-<storage-operation>/resourceId=/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<source-storage-account>/blobServices/default/y=<year>/m=<month>/d=<day>/h=<hour>/m=<minute>/PT1H.json`

Voici un exemple :

`https://mylogstorageaccount.blob.core.windows.net/insights-logs-storagewrite/resourceId=/subscriptions/`<br>`208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/blobServices/default/y=2019/m=07/d=30/h=23/m=12/PT1H.json`

### <a name="accessing-logs-in-an-event-hub"></a>Accès aux journaux dans un hub d’événements

Les journaux envoyés à un hub d’événements ne sont pas stockés en tant que fichier, mais vous pouvez vérifier que le hub d’événements a reçu les informations du journal. Dans le Portail Azure, accédez à votre Event Hub et vérifiez que le nombre de **messages entrants** est supérieur à zéro. 

![Journaux d’audit](media/monitor-blob-storage/event-hub-log.png)

Vous pouvez accéder aux données de journal envoyées à votre hub d’événements et les lire à l’aide d’outils d’analyse et de gestion d’événements et d’informations de sécurité. Pour plus d’informations, consultez [Que faire avec les données de supervision envoyées à mon hub d’événements ?](../../azure-monitor/platform/stream-monitoring-data-event-hubs.md).

### <a name="accessing-logs-in-a-log-analytics-workspace"></a>Accès aux journaux dans un espace de travail Log Analytics

Vous pouvez accéder aux journaux envoyés à un espace de travail Log Analytics en utilisant des requêtes de journal Azure Monitor.

Pour plus d’informations, voir [Bien démarrer avec Log Analytics dans Azure Monitor](../../azure-monitor/log-query/log-analytics-tutorial.md).

Les données sont stockées dans la table **StorageBlobLog**. Les journaux de Data Lake Storage Gen2 n’apparaissent pas dans une table dédiée. Cela est dû au fait que Data Lake Storage Gen2 n’est pas un service. Il s’agit d’un ensemble de fonctionnalités que vous pouvez activer dans votre compte de stockage. Si vous avez activé ces fonctionnalités, les journaux continueront d’apparaître dans la table StorageBlobLogs. 

#### <a name="sample-kusto-queries"></a>Exemples de requêtes Kusto

Voici quelques requêtes que vous pouvez entrer dans la barre **Recherche dans les journaux** pour vous permettre de superviser votre service Stockage Blob. Ces requêtes fonctionnent avec le [nouveau langage](../../azure-monitor/log-query/log-query-overview.md).

> [!IMPORTANT]
> Quand vous sélectionnez **Journaux** dans le menu du groupe de ressources du compte de stockage, Log Analytics est ouvert avec l’étendue de requête définie sur le groupe de ressources actuel. Cela signifie que les requêtes de journal n’incluront que les données de ce groupe de ressources. Si vous voulez exécuter une requête qui inclut des données provenant d’autres ressources ou d’autres services Azure, sélectionnez **Journaux** dans le menu **Azure Monitor**. Pour plus d’informations, consultez [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](../../azure-monitor/log-query/scope.md).

Utilisez ces requêtes pour mieux superviser vos comptes de Stockage Azure :

* Pour lister les 10 erreurs les plus courantes au cours des trois derniers jours.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by StatusText
    | top 10 by count_ desc
    ```
* Pour lister les 10 principales opérations qui ont provoqué le plus d’erreurs au cours des trois derniers jours.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by OperationName
    | top 10 by count_ desc
    ```
* Pour lister les 10 opérations ayant la latence de bout en bout la plus longue au cours des trois derniers jours.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d)
    | top 10 by DurationMs desc
    | project TimeGenerated, OperationName, DurationMs, ServerLatencyMs, ClientLatencyMs = DurationMs - ServerLatencyMs
    ```
* Pour lister toutes les opérations ayant provoqué des erreurs de limitation côté serveur au cours des trois derniers jours.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and StatusText contains "ServerBusy"
    | project TimeGenerated, OperationName, StatusCode, StatusText
    ```
* Pour lister toutes les demandes avec un accès anonyme au cours des trois derniers jours.

    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and AuthenticationType == "Anonymous"
    | project TimeGenerated, OperationName, AuthenticationType, Uri
    ```
* Pour créer un graphique à secteurs des opérations utilisées au cours des trois derniers jours.
    ```Kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d)
    | summarize count() by OperationName
    | sort by count_ desc 
    | render piechart
    ```
## <a name="faq"></a>Questions fréquentes (FAQ)

**Le stockage Azure prend-il en charge les métriques de disques managés ou de disques non managés ?**

Non. Azure Compute prend en charge les métriques sur les disques. Pour obtenir davantage d’informations, consultez [Métriques par disque pour les disques managés et non managés](https://azure.microsoft.com/blog/per-disk-metrics-managed-disks/).

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des informations de référence sur les journaux et les métriques créés par le Stockage Blob Azure, consultez [Informations de référence sur les données de supervision du Stockage Blob Azure](monitor-blob-storage-reference.md).
- Pour plus d’informations sur la supervision des ressources Azure, consultez [Superviser des ressources Azure avec Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
- Pour plus d’informations sur la migration des métriques, consultez [Migration des métriques de Stockage Azure](../common/storage-metrics-migration.md).