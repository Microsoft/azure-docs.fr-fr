---
title: Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations
description: Envoyez les journaux et métriques de plateforme Azure Monitor vers les journaux d’activité Azure Monitor, le stockage Azure ou Azure Event Hubs à l’aide d’un paramètre de diagnostic.
author: bwren
ms.author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 04/27/2020
ms.subservice: logs
ms.openlocfilehash: 950fcdc5cd6a5bbf3fa61ebd5e23be89691c4370
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95535790"
---
# <a name="create-diagnostic-settings-to-send-platform-logs-and-metrics-to-different-destinations"></a>Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations
Les [journaux de plateforme](platform-logs-overview.md) dans Azure, y compris le journal d’activité Azure et les journaux de ressources, fournissent des informations de diagnostic et d’audit détaillées pour les ressources Azure et la plateforme Azure dont elles dépendent. Les [métriques de plateforme](data-platform-metrics.md) sont collectées par défaut et généralement stockées dans la base de données de métriques Azure Monitor. Cet article fournit des détails sur la création et la configuration de paramètres de diagnostic pour envoyer les journaux de plateforme et les métriques de plateforme vers différentes destinations.

> [!IMPORTANT]
> Avant de créer un paramètre de diagnostic pour le journal d’activité, vous devez d’abord désactiver toute configuration héritée. Pour plus d’informations, consultez [Méthodes de collecte héritées](activity-log.md#legacy-collection-methods).

Chaque ressource Azure requiert son propre paramètre de diagnostic, qui définit les critères suivants :

- Catégories de journaux et données de métriques envoyées aux destinations définies dans le paramètre. Les catégories disponibles varient en fonction des types de ressources.
- Une ou plusieurs destinations auxquelles envoyer les journaux. Les destinations actuelles sont l’espace de travail Log Analytics, Event Hubs et le Stockage Azure.

Un seul paramètre de diagnostic ne peut pas définir plus d’une destination. Si vous souhaitez envoyer des données à plus d’un type de destination (par exemple, deux espaces de travail Log Analytics), créez plusieurs paramètres. Chaque ressource peut avoir jusqu’à 5 paramètres de diagnostic.

La vidéo suivante vous guide dans l’acheminement des journaux de plateforme avec les paramètres de diagnostic.
> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4AvVO]

> [!NOTE]
> Les [métriques de plateforme](metrics-supported.md) sont envoyées automatiquement vers les [métriques Azure Monitor](data-platform-metrics.md). Vous pouvez utiliser des paramètres de diagnostic pour envoyer des métriques pour certains services Azure vers des journaux Azure Monitor à des fins d’analyse avec d’autres données de supervision en utilisant des [requêtes de journal](../log-query/log-query-overview.md) dans certaines limites. 
>  
>  
> L’envoi de métriques multidimensionnelles via les paramètres de diagnostic n’est pas pris en charge actuellement. Les métriques à plusieurs dimensions sont exportées en tant que métriques dimensionnelles uniques aplaties, puis agrégées dans les valeurs de la dimension. *Par exemple* : La métrique « IOReadBytes » sur un blockchain peut être explorée et représentée sur un graphique par niveau de nœud. Toutefois, lorsque la métrique est exportée par le biais de paramètres de diagnostic, elle représente tous les octets lus pour tous les nœuds. En outre, en raison de limitations internes, certaines métriques ne sont pas exportables vers des journaux Azure Monitor ou Log Analytics. Pour plus d’informations, consultez la [liste des mesures exportables](metrics-supported-export-diagnostic-settings.md). 
>  
>  
> Pour contourner ces limitations pour des mesures spécifiques, nous vous suggérons d’extraire celles-ci manuellement à l’aide de l’[API REST Métriques](/rest/api/monitor/metrics/list) et de les importer dans des journaux Azure Monitor à l’aide de l’[API Collecteur de données d’Azure Monitor](data-collector-api.md).  


## <a name="destinations"></a>Destinations
Il est possible d’envoyer des journaux et métriques de plateforme aux destinations indiquées dans le tableau suivant. 

| Destination | Description |
|:---|:---|
| [Espace de travail Log Analytics](design-logs-deployment.md) | L’envoi de journaux et de métriques vers un espace de travail Log Analytics vous permet de les analyser avec d’autres données de supervision collectées par Azure Monitor à l’aide de requêtes de journal puissantes, ainsi que d’exploiter d’autres fonctionnalités d’Azure Monitor, telles que les alertes et les visualisations. |
| [Event Hubs](../../event-hubs/index.yml) | L’envoi de journaux et de métriques à Event Hubs vous permet de transmettre en continu des données à des systèmes externes tels que des solutions SIEM tierces et d’autres solutions d’analytique des journaux d’activité.  |
| [Compte Azure Storage](../../storage/blobs/index.yml) | L’archivage des journaux et des métriques dans un compte de stockage Azure est utile à des fins d’audit, d’analyse statique ou de sauvegarde. Par rapport aux journaux d’activité Azure Monitor et à l’espace de travail Log Analytics, le stockage Azure est moins onéreux et les journaux peuvent y être conservés indéfiniment.  |


### <a name="destination-requirements"></a>Exigences de destination

Les destinations du paramètre de diagnostic doivent être créées avant les paramètres de diagnostic. Cette destination ne doit pas nécessairement se trouver dans le même abonnement que la ressource qui envoie des journaux, à condition que l’utilisateur qui configure le paramètre ait un accès Azure RBAC approprié aux deux abonnements. Le tableau suivant fournit des exigences uniques pour chaque destination, y compris des restrictions régionales.

| Destination | Spécifications |
|:---|:---|
| Espace de travail Log Analytics | Il n’est pas obligatoire que l’espace de travail réside dans la même région que la ressource supervisée.|
| Hubs d'événements | La stratégie d’accès partagé pour l’espace de noms définit les autorisations dont dispose le mécanisme de diffusion en continu. La diffusion en continu vers Event Hubs requiert des autorisations de gestion, d’envoi et d’écoute. Pour mettre à jour le paramètre de diagnostic afin d’inclure la diffusion en continu, vous devez disposer de l’autorisation ListKey sur la règle d’autorisation Event Hubs.<br><br>L’espace de noms Event Hub doit se trouver dans la même région que la ressource supervisée si la ressource est régionale. |
| Compte Azure Storage | Nous vous déconseillons d'utiliser un compte de stockage existant sur lequel sont stockées d’autres données de non-analyse, afin de pouvoir mieux contrôler l’accès aux données. Si vous archivez également des journaux de diagnostic et des journaux de ressources ensemble, vous pouvez choisir d’utiliser le même compte de stockage pour regrouper toutes vos données d’analyse au même emplacement.<br><br>Pour envoyer les données dans l'espace de stockage immuable, définissez la stratégie d'immuabilité du compte de stockage, comme décrit dans [Définir et gérer des stratégies d’immuabilité pour le stockage d'objets blob](../../storage/blobs/storage-blob-immutability-policies-manage.md). Vous devez suivre toutes les étapes décrites dans cet article, y compris l’activation des écritures protégées d'objets blob d'ajout.<br><br>Le compte de stockage doit se trouver dans la même région que la ressource supervisée si la ressource est régionale. |

> [!NOTE]
> Actuellement, les comptes ADLS Gen2 ne sont pas pris en charge en tant que destination pour les paramètres de diagnostic, même s’ils peuvent être répertoriés comme une option valide dans le portail Azure.

> [!NOTE]
> Azure Monitor (ses paramètres de diagnostic) ne peut pas accéder aux ressources Event Hubs lorsque les réseaux virtuels sont activés. Vous devez activer le paramètre Autoriser les services Microsoft approuvés à contourner ce pare-feu dans Event Hub, afin que le service Azure Monitor (paramètres de diagnostic) soit autorisé à accéder à vos ressources Event Hubs. 


## <a name="create-in-azure-portal"></a>Créer dans le portail Azure

Vous pouvez configurer des paramètres de diagnostic sur le portail Azure à partir du menu Azure Monitor ou du menu de la ressource.

1. L’endroit où vous configurez les paramètres de diagnostic dans le portail Azure dépend de la ressource.

   - Pour une ressource unique, cliquez sur **Paramètres de diagnostic** sous **Analyse** dans le menu de la ressource.

        ![Capture d’écran de la section Surveillance d’un menu de ressources dans le portail Azure avec les Paramètres de diagnostic en surbrillance.](media/diagnostic-settings/menu-resource.png)

   - Pour une ou plusieurs ressources, dans le menu Azure Monitor, sous **Paramètres**, cliquez sur **Paramètres de diagnostic**, puis sur la ressource.

        ![Capture d’écran de la section Paramètres dans le menu d’Azure Monitor avec les Paramètres de diagnostic en surbrillance.](media/diagnostic-settings/menu-monitor.png)

   - Pour le journal d’activité, dans le menu **Azure Monitor**, cliquez sur **Journal d’activité**, puis sur **Paramètres de diagnostic**. Veillez à désactiver toute configuration héritée pour le journal d’activité. Pour plus d’informations, voir [Désactiver des paramètres existants](./activity-log.md#legacy-collection-methods).

        ![Capture d’écran du menu d’Azure Monitor avec le Journal d’activité sélectionné et les Paramètres de diagnostic en surbrillance dans la barre de menus du journal d’activité.](media/diagnostic-settings/menu-activity-log.png)

2. S’il n’existe aucun paramètre sur la ressource que vous avez sélectionnée, vous êtes invité à créer un paramètre. Cliquez sur **Ajouter le paramètre de diagnostic**.

   ![Ajouter le paramètre de diagnostic - aucun paramètre existant](media/diagnostic-settings/add-setting.png)

   Si des paramètres sont définis pour la ressource, la liste des paramètres déjà configurés s’affiche. Cliquez sur **Ajouter un paramètre de diagnostic** pour ajouter un paramètre ou sur **Modifier le paramètre** pour modifier un paramètre existant. Chaque paramètre ne peut pas avoir plus d’un type de destination.

   ![Ajouter le paramètre de diagnostic - paramètres existants](media/diagnostic-settings/edit-setting.png)

3. Si votre paramètre n’a pas de nom, nommez-le.

    ![Ajouter un paramètre de diagnostic](media/diagnostic-settings/setting-new-blank.png)

4. **Détails des catégories (éléments à acheminer)**  : cochez la case pour chaque catégorie de données que vous voulez envoyer aux destinations spécifiées plus tard. La liste des catégories varie pour chaque service Azure.

     - **AllMetrics** achemine les métriques de plateforme d’une ressource dans le magasin Azure Logs, mais sous forme de journal. Ces métriques sont généralement envoyées uniquement à la base de données de série chronologique des métriques d’Azure Monitor. Les envoyer au magasin Azure Monitor Logs (qui peut faire l’objet d’une recherche via Log Analytics) vous aide à les intégrer dans des requêtes qui effectuent des recherches dans d’autres journaux. Cette option peut ne pas être disponible pour tous les types de ressources. Lorsqu’elle est prise en charge, la catégorie [Métriques prises en charge par Azure Monitor](metrics-supported.md) répertorie les métriques collectées pour les types de ressources.

       > [!NOTE]
       > Voir la limitation relative à l’acheminement de métriques vers les journaux Azure Monitor, plus haut dans cet article.  


     - **Journaux** répertorie les différentes catégories disponibles en fonction du type de ressource. Cochez chaque catégorie que vous souhaitez acheminer vers une destination.

5. **Détails de la destination** : cochez la case pour chaque destination. Lorsque vous cochez chaque case, des options s’affichent pour vous permettre d’ajouter des informations supplémentaires.

      ![Envoyer à Log Analytics ou à Event Hubs](media/diagnostic-settings/send-to-log-analytics-event-hubs.png)

    1. **Log Analytics** : entrez l’abonnement et l’espace de travail.  Si vous n’avez pas d’espace de travail, vous devez [en créer un avant de continuer](../learn/quick-create-workspace.md).

    1. **Event Hubs** : spécifiez les critères suivants :
       - Abonnement auquel appartient l’Event Hub.
       - Espace de noms Event Hub : si vous n’en avez pas encore, vous devez [en créer un](../../event-hubs/event-hubs-create.md).
       - Nom Event Hub (facultatif) auquel envoyer toutes les données. Si vous ne spécifiez pas de nom, un hub d’événements est créé pour chaque catégorie de journal. Si vous envoyez plusieurs catégories, vous pouvez spécifier un nom pour limiter le nombre de hubs d’événements créés. Pour plus de détails, voir [Quotas et limites d’Azure Event Hubs](../../event-hubs/event-hubs-quotas.md).
       - Stratégie Event Hub (facultatif) : une stratégie définit les autorisations dont dispose le mécanisme de diffusion en continu. Pour plus d’informations, consultez [Event-hubs-features](../../event-hubs/event-hubs-features.md#publisher-policy).

    1. **Stockage** : choisissez l’abonnement, le compte de stockage et la stratégie de rétention.

        ![Envoyer à Stockage](media/diagnostic-settings/storage-settings-new.png)

        > [!TIP]
        > Envisagez de définir la stratégie de rétention sur 0 et de supprimer manuellement vos données du stockage à l’aide d’une tâche planifiée afin d’éviter toute confusion à l’avenir.
        >
        > Tout d’abord, si vous utilisez le stockage pour l’archivage, vous souhaitez généralement que vos données soient préservées plus de 365 jours. Ensuite, si vous choisissez une stratégie de rétention supérieure à 0, la date d’expiration est jointe aux journaux au moment du stockage. Vous ne pouvez pas modifier la date de ces journaux une fois qu’ils sont stockés.
        >
        > Par exemple, si vous définissez la stratégie de rétention pour *WorkflowRuntime* sur 180 jours, puis que vous la définissez 24 heures plus tard sur 365 jours, les journaux stockés pendant ces 24 heures sont automatiquement supprimés après 180 jours, tandis que tous les journaux suivants de ce type seront automatiquement supprimés après 365 jours. La modification ultérieure de la stratégie de rétention ne permet pas de conserver les journaux des premières 24 heures pendant 365 jours.

6. Cliquez sur **Enregistrer**.

Après quelques instants, le nouveau paramètre apparaît dans la liste des paramètres de cette ressource, et les journaux sont envoyés aux destinations spécifiées au fur et à mesure de la génération de nouvelles données d’événement. Un délai de 15 minutes peut s’écouler entre le moment où un événement est émis et celui où il [s’affiche dans un espace de travail Log Analytics](data-ingestion-time.md).

## <a name="create-using-powershell"></a>Créer à l’aide de PowerShell

Pour créer un paramètre de diagnostic avec [Azure PowerShell](../samples/powershell-samples.md), utilisez la cmdlet [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting). Pour obtenir une description des paramètres de celle-ci, consultez sa documentation.

> [!IMPORTANT]
> Vous ne pouvez pas utiliser cette méthode pour le journal d’activité Azure. Utilisez plutôt la méthode [Créer un paramètre de diagnostic dans Azure Monitor à l’aide d’un modèle Resource Manager](../samples/resource-manager-diagnostic-settings.md) pour créer un modèle Resource Manager et le déployer avec PowerShell.

Voici un exemple de cmdlet PowerShell permettant de créer un paramètre de diagnostic utilisant les trois destinations.

```powershell
Set-AzDiagnosticSetting -Name KeyVault-Diagnostics -ResourceId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault -Category AuditEvent -MetricCategory AllMetrics -Enabled $true -StorageAccountId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount -WorkspaceId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/myworkspace  -EventHubAuthorizationRuleId /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey
```

## <a name="create-using-azure-cli"></a>Créer à l’aide d’Azure CLI

Pour créer un paramètre de diagnostic avec [Azure CLI](/cli/azure/monitor?view=azure-cli-latest), utilisez la commande [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest#az-monitor-diagnostic-settings-create). Pour une description des paramètres de cette commande, consultez sa documentation.

> [!IMPORTANT]
> Vous ne pouvez pas utiliser cette méthode pour le journal d’activité Azure. Utilisez plutôt la méthode [Créer un paramètre de diagnostic dans Azure Monitor à l’aide d’un modèle Resource Manager](../samples/resource-manager-diagnostic-settings.md) pour créer un modèle Resource Manager et le déployer avec CLI.

Voici un exemple de commande Azure CLI pour créer un paramètre de diagnostic utilisant les trois destinations.

```azurecli
az monitor diagnostic-settings create  \
--name KeyVault-Diagnostics \
--resource /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault \
--logs    '[{"category": "AuditEvent","enabled": true}]' \
--metrics '[{"category": "AllMetrics","enabled": true}]' \
--storage-account /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
--workspace /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/myworkspace \
--event-hub-rule /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey
```

## <a name="create-using-resource-manager-template"></a>Créer à l’aide d’un modèle Resource Manager
Consultez [Exemples de modèle Resource Manager pour les paramètres de diagnostic dans Azure Monitor](../samples/resource-manager-diagnostic-settings.md) pour créer ou mettre à jour des paramètres de diagnostic avec un modèle Resource Manager.

## <a name="create-using-rest-api"></a>Créer à l’aide de l’API REST
Pour créer ou mettre à jour des paramètres de diagnostic à l’aide de l’[API REST Azure Monitor](/rest/api/monitor/), voir [Paramètres de diagnostic](/rest/api/monitor/diagnosticsettings).

## <a name="create-using-azure-policy"></a>Créer à l’aide d’Azure Policy
Sachant qu’un paramètre de diagnostic doit être créé pour chaque ressource Azure, vous pouvez utiliser Azure Policy pour qu’un paramètre de diagnostic soit créé automatiquement à chaque création de ressource. Pour plus d’informations, consultez [Déployer Azure Monitor à la bonne échelle à l’aide d’Azure Policy](../deploy-scale.md).


## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur les journaux de la plateforme Azure](platform-logs-overview.md)
