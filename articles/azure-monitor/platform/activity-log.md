---
title: Journal d’activité Azure
description: Affichez le journal d’activité Azure et envoyez-le aux journaux d’activité d’Azure Monitor, à Azure Event Hubs et à Stockage Azure.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 06/12/2020
ms.author: bwren
ms.subservice: logs
ms.openlocfilehash: 1b49faabb1c61a10418bfce3ae2e8187429981ad
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186080"
---
# <a name="azure-activity-log"></a>Journal d’activité Azure
Le journal d’activité est un [journal de plateforme](platform-logs-overview.md) dans Azure qui fournit un aperçu de tous les événements de niveau d’abonnement. Les informations qu’il contient indiquent par exemple à quel moment une ressource a été modifiée ou une machine virtuelle a été démarrée. Vous pouvez afficher le journal d’activité dans le portail Azure ou récupérer des entrées avec PowerShell et l’interface CLI. Pour obtenir des fonctionnalités supplémentaires, vous devez créer un paramètre de diagnostic permettant d’envoyer le journal d’activité aux [journaux d’activité d’Azure Monitor](data-platform-logs.md), à Azure Event Hubs pour le transférer en dehors d’Azure ou à Stockage Azure à des fins d’archivage. Cet article fournit des détails sur l’affichage du journal d’activité et son envoi vers différentes destinations.

Pour plus d’informations sur la création d’un paramètre de diagnostic, consultez [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](diagnostic-settings.md).

> [!NOTE]
> Les entrées du journal d’activité sont générées par le système et ne peuvent pas être modifiées ou supprimées.

## <a name="view-the-activity-log"></a>Afficher le journal d’activité
Vous pouvez accéder au journal d’activité à partir de la plupart des menus du portail Azure. Le menu à partir duquel vous l’ouvrez détermine son filtre initial. Si vous l’ouvrez à partir du menu **Monitor**, le seul filtre sera sur l’abonnement. Si vous l’ouvrez à partir du menu d’une ressource, le filtre est défini sur cette ressource. Vous pouvez toujours modifier le filtre pour afficher toutes les autres entrées. Cliquez sur **Ajouter un filtre** pour ajouter des propriétés supplémentaires au filtre.

![Afficher le journal d’activité](./media/activity-logs-overview/view-activity-log.png)

Pour obtenir une description des catégories du journal d’activité, consultez [Schéma d’événements du journal d’activité](activity-log-schema.md#categories).

### <a name="view-change-history"></a>Afficher l'historique des modifications

Pour certains événements, vous pouvez afficher l’historique des modifications, qui indique les modifications qui sont survenues au cours de cette période. Sélectionnez un événement du journal d’activité sur lequel vous souhaitez vous pencher. Sélectionnez l'onglet **Historique des modifications (préversion)** pour afficher les modifications associées à cet événement.

![Historique des modifications pour un événement](media/activity-logs-overview/change-history-event.png)

Si des modifications ont été apportées à l’événement, la liste des modifications que vous pouvez sélectionner s'affiche. Vous accédez à la page **Historique des modifications (préversion)** . Les modifications apportées à la ressource s'affichent sur cette page. Dans l’exemple suivant, vous pouvez voir non seulement que la taille de la machine virtuelle a changé, mais aussi sa taille avant et après la modification. Pour en savoir plus sur l’historique des modifications, consultez [Obtenir les modifications des ressources](../../governance/resource-graph/how-to/get-resource-changes.md).

![Page Historique des modifications affichant les différences](media/activity-logs-overview/change-history-event-details.png)


### <a name="other-methods-to-retrieve-activity-log-events"></a>Autres méthodes pour récupérer les événements du journal d’activité
Vous pouvez également accéder aux événements du journal d’activité à l’aide des méthodes suivantes.

- Utilisez la cmdlet [Get-AzLog](/powershell/module/az.monitor/get-azlog) pour récupérer le journal d’activité à partir de PowerShell. Consultez [Exemples PowerShell Azure Monitor](../samples/powershell-samples.md#retrieve-activity-log).
- Utilisez [az monitor activity-log](/cli/azure/monitor/activity-log) pour récupérer le journal d’activité à partir de l’interface CLI.  Consultez les [exemples d’interface de ligne de commande Azure Monitor](../samples/cli-samples.md#view-activity-log).
- Utilisez l'[API REST Azure Monitor](/rest/api/monitor/) pour récupérer le journal d’activité à partir d’un client REST. 


## <a name="send-to-log-analytics-workspace"></a>Envoyer à l’espace de travail Log Analytics
 Envoyez le journal d’activité à un espace de travail Log Analytics pour activer les fonctionnalités des [journaux d’activité Azure Monitor](data-platform-logs.md), qui permettent notamment de :

- Mettre en corrélation les données du journal d’activité avec d’autres données d’analyse collectées par Azure Monitor.
- Consolider les entrées de journal de plusieurs abonnements et locataires Azure en un seul endroit pour les analyser ensemble.
- Utiliser les requêtes de journal pour effectuer des analyses complexes et obtenir des informations détaillées sur les entrées du journal d’activité.
- Utiliser des alertes de journal avec des entrées d’activité qui permettent une logique d’alerte plus complexe.
- Stocker les entrées du journal d’activité au-delà de 90 jours.
- Aucune ingestion des données ni aucuns frais de rétention pour les données de journal d’activité stockées dans un espace de travail Log Analytics.

[Créez un paramètre de diagnostic](diagnostic-settings.md) pour envoyer le journal d’activité à un espace de travail Log Analytics. Vous pouvez envoyer le journal d’activité d’un abonnement unique vers un maximum de cinq espaces de travail. La collecte des journaux de plusieurs locataires nécessite [Azure Lighthouse](../../lighthouse/index.yml).

Les données du journal d’activité d’un espace de travail Log Analytics sont stockées dans une table appelée *AzureActivity* que vous pouvez récupérer à l’aide d’une [requête de journal](../log-query/log-query-overview.md) dans [Log Analytics](../log-query/log-analytics-tutorial.md). La structure de cette table varie selon la [catégorie de l’entrée de journal](activity-log-schema.md). Pour obtenir une description des propriétés de la table, consultez la [référence des données d’Azure Monitor](/azure/azure-monitor/reference/tables/azureactivity).

Par exemple, pour afficher le nombre d’enregistrements du journal d’activité pour chaque catégorie, utilisez la requête suivante.

```kusto
AzureActivity
| summarize count() by Category
```

Pour récupérer tous les enregistrements de la catégorie administrative, utilisez la requête suivante.

```kusto
AzureActivity
| where Category == "Administrative"
```


## <a name="send-to-azure-event-hubs"></a>Envoyer à Azure Event Hubs
Envoyez le journal d’activité à Azure Event Hubs pour envoyer les entrées en dehors d’Azure, par exemple à la solution SIEM d’un tiers ou à d’autres solutions d’analytique des journaux d’activité. Les événements du journal d’activité en provenance d’Event Hubs sont consommés au format JSON avec un élément `records` contenant les enregistrements de chaque charge utile. Le schéma dépend de la catégorie et est décrit dans [Schéma à partir du compte de stockage et des Event Hubs](activity-log-schema.md).

Voici des exemples de données de sortie provenant d’Event Hubs pour un journal d’activité :

``` JSON
{
    "records": [
        {
            "time": "2019-01-21T22:14:26.9792776Z",
            "resourceId": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
            "operationName": "microsoft.support/supporttickets/write",
            "category": "Write",
            "resultType": "Success",
            "resultSignature": "Succeeded.Created",
            "durationMs": 2826,
            "callerIpAddress": "111.111.111.11",
            "correlationId": "c776f9f4-36e5-4e0e-809b-c9b3c3fb62a8",
            "identity": {
                "authorization": {
                    "scope": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
                    "action": "microsoft.support/supporttickets/write",
                    "evidence": {
                        "role": "Subscription Admin"
                    }
                },
                "claims": {
                    "aud": "https://management.core.windows.net/",
                    "iss": "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
                    "iat": "1421876371",
                    "nbf": "1421876371",
                    "exp": "1421880271",
                    "ver": "1.0",
                    "http://schemas.microsoft.com/identity/claims/tenantid": "00000000-0000-0000-0000-000000000000",
                    "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
                    "http://schemas.microsoft.com/identity/claims/objectidentifier": "2468adf0-8211-44e3-95xq-85137af64708",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "admin@contoso.com",
                    "puid": "20030000801A118C",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "9vckmEGF7zDKk1YzIY8k0t1_EAPaXoeHyPRn6f413zM",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "John",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Smith",
                    "name": "John Smith",
                    "groups": "cacfe77c-e058-4712-83qw-f9b08849fd60,7f71d11d-4c41-4b23-99d2-d32ce7aa621c,31522864-0578-4ea0-9gdc-e66cc564d18c",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": " admin@contoso.com",
                    "appid": "c44b4083-3bq0-49c1-b47d-974e53cbdf3c",
                    "appidacr": "2",
                    "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
                    "http://schemas.microsoft.com/claims/authnclassreference": "1"
                }
            },
            "level": "Information",
            "location": "global",
            "properties": {
                "statusCode": "Created",
                "serviceRequestId": "50d5cddb-8ca0-47ad-9b80-6cde2207f97c"
            }
        }
    ]
}
```


## <a name="send-to--azure-storage"></a>Envoyer à Stockage Azure
Envoyez le journal d’activité à un compte de stockage Azure si vous souhaitez conserver vos données de journal plus de 90 jours à des fins d’audit, d’analyse statique ou de sauvegarde. Si vous devez conserver vos événements pendant 90 jours ou moins, il est inutile de configurer l’archivage sur un compte de stockage, puisque les événements du journal d’activité sont conservés dans la plateforme Azure pendant 90 jours.

Lorsque vous envoyez le journal d’activité à Azure, un conteneur de stockage est créé dans le compte de stockage dès qu’un événement se produit. Les blobs présents dans le conteneur utilisent la convention d’affectation de noms suivante :

```
insights-activity-logs/resourceId=/SUBSCRIPTIONS/{subscription ID}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Par exemple, un blob particulier peut avoir un nom similaire à ce qui suit :

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/y=2020/m=06/d=08/h=18/m=00/PT1H.json
```

Chaque objet blob PT1H.json contient un objet blob d’événements JSON qui se sont produits pendant l’heure spécifiée dans l’URL de l’objet blob (par exemple, h=12). Pendant l’heure en cours, les événements sont ajoutés au fichier PT1H.json à mesure qu’ils se produisent. La valeur de minute (m=00) est toujours 00, car les événements du journal de ressource sont divisés en objets blob par heure.

Chaque événement est stocké dans le fichier PT1H.json au format suivant qui utilise un schéma de niveau supérieur commun, mais qui est autrement unique pour chaque catégorie, comme décrit dans [Schéma du journal d’activité](activity-log-schema.md).

``` JSON
{ "time": "2020-06-12T13:07:46.766Z", "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MV-VM-01", "correlationId": "0f0cb6b4-804b-4129-b893-70aeeb63997e", "operationName": "Microsoft.Resourcehealth/healthevent/Updated/action", "level": "Information", "resultType": "Updated", "category": "ResourceHealth", "properties": {"eventCategory":"ResourceHealth","eventProperties":{"title":"This virtual machine is starting as requested by an authorized user or process. It will be online shortly.","details":"VirtualMachineStartInitiatedByControlPlane","currentHealthStatus":"Unknown","previousHealthStatus":"Unknown","type":"Downtime","cause":"UserInitiated"}}}
```


## <a name="legacy-collection-methods"></a>Méthodes de collecte héritées
Cette section décrit les méthodes héritées visant à collecter le journal d’activité qui étaient utilisées avant les paramètres de diagnostic. Si vous utilisez ces méthodes, vous devriez envisager de passer aux paramètres de diagnostic qui offrent une meilleure fonctionnalité et une meilleure cohérence avec les journaux de ressources.

### <a name="log-profiles"></a>Profils de journal
Les profils de journal sont la méthode héritée pour l’envoi du journal d’activité vers le stockage Azure ou les Event Hubs. Utilisez la procédure suivante pour continuer à utiliser un profil de journal ou pour le désactiver en vue de la migration vers un paramètre de diagnostic.

1. À partir du menu **Azure Monitor** du Portail Azure, sélectionnez **Journal d’activité**.
3. Cliquez sur **Paramètres de diagnostic**.

   ![Paramètres de diagnostic](media/diagnostic-settings-subscription/diagnostic-settings.png)

4. Cliquez sur la bannière violette pour obtenir l’expérience héritée.

    ![Expérience héritée](media/diagnostic-settings-subscription/legacy-experience.png)



### <a name="configure-log-profile-using-powershell"></a>Configurer le profil de journal à l’aide de PowerShell

Si un profil de journal existe déjà, vous devez tout d’abord le supprimer, puis en créer un nouveau.

1. Utilisez `Get-AzLogProfile` pour déterminer s’il existe un profil de journal.  S’il n’existe aucun profil de journal, notez la propriété *name*.

1. Utilisez `Remove-AzLogProfile` pour supprimer le profil de journal à l’aide de la valeur provenant de la propriété *name*.

    ```powershell
    # For example, if the log profile name is 'default'
    Remove-AzLogProfile -Name "default"
    ```

3. Utilisez `Add-AzLogProfile` pour créer un profil de journal :

    ```powershell
    Add-AzLogProfile -Name my_log_profile -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Location global,westus,eastus -RetentionInDays 90 -Category Write,Delete,Action
    ```

    | Propriété | Obligatoire | Description |
    | --- | --- | --- |
    | Nom |Oui |Nom de votre profil de journal. |
    | StorageAccountId |Non |ID de ressource du compte de stockage dans lequel le journal d’activité doit être enregistré. |
    | serviceBusRuleId |Non |ID de règle Service Bus pour l’espace de noms Service Bus dans lequel vous souhaitez que des concentrateurs d’événements soient créés. Il s'agit d'une chaîne au format `{service bus resource ID}/authorizationrules/{key name}`. |
    | Emplacement |Oui |Liste séparée par des virgules des régions pour lesquelles vous souhaitez collecter les événements du journal d’activité. |
    | RetentionInDays |Oui |Nombre de jours pendant lesquels les événements doivent être conservés dans le compte de stockage, compris entre 1 et 365. Une valeur de zéro signifie que les journaux d’activité seront stockés pour une durée indéfinie. |
    | Category |Non |Liste séparée par des virgules des catégories d’événements qui doivent être collectées. Les valeurs possibles sont _Write_, _Delete_ et _Action_. |

### <a name="example-script"></a>Exemple de script
Vous trouverez ci-dessous un exemple de script PowerShell pour créer un profil de journal qui écrit le journal d’activité dans un compte de stockage et un Event Hub.

   ```powershell
   # Settings needed for the new log profile
   $logProfileName = "default"
   $locations = (Get-AzLocation).Location
   $locations += "global"
   $subscriptionId = "<your Azure subscription Id>"
   $resourceGroupName = "<resource group name your event hub belongs to>"
   $eventHubNamespace = "<event hub namespace>"

   # Build the service bus rule Id from the settings above
   $serviceBusRuleId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.EventHub/namespaces/$eventHubNamespace/authorizationrules/RootManageSharedAccessKey"

   # Build the storage account Id from the settings above
   $storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

   Add-AzLogProfile -Name $logProfileName -Location $locations -StorageAccountId  $storageAccountId -ServiceBusRuleId $serviceBusRuleId
   ```


### <a name="configure-log-profile-using-azure-cli"></a>Configurer le profil de journal à l’aide d’Azure CLI

Si un profil de journal existe déjà, vous devez tout d’abord le supprimer, puis créer un nouveau profil de journal.

1. Utilisez `az monitor log-profiles list` pour déterminer s’il existe un profil de journal.
2. Utilisez `az monitor log-profiles delete --name "<log profile name>` pour supprimer le profil de journal à l’aide de la valeur provenant de la propriété *name*.
3. Utilisez `az monitor log-profiles create` pour créer un profil de journal :

   ```azurecli-interactive
   az monitor log-profiles create --name "default" --location null --locations "global" "eastus" "westus" --categories "Delete" "Write" "Action"  --enabled false --days 0 --service-bus-rule-id "/subscriptions/<YOUR SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUB NAME SPACE>/authorizationrules/RootManageSharedAccessKey"
   ```

    | Propriété | Obligatoire | Description |
    | --- | --- | --- |
    | name |Oui |Nom de votre profil de journal. |
    | storage-account-id |Oui |ID de ressource du compte de stockage dans lequel les journaux d’activité doivent être enregistrés. |
    | locations |Oui |Liste, séparée par des espaces, des régions pour lesquelles vous souhaitez collecter les événements du journal d’activité. Vous pouvez voir une liste de toutes les régions pour votre abonnement à l’aide de `az account list-locations --query [].name`. |
    | jours |Oui |Nombre de jours pendant lesquels les événements doivent être conservés, compris entre 1 et 365. Une valeur de zéro signifie que les journaux d’activité seront stockés pour une durée indéfinie (pour toujours).  Si zéro est spécifié, le paramètre enabled doit être défini sur false. |
    |enabled | Oui |True ou False.  Utilisée pour activer ou désactiver la stratégie de rétention.  Si la valeur est True, le paramètre days doit être une valeur supérieure à 0.
    | categories |Oui |Liste, séparée par des espaces, des catégories d’événements qui doivent être collectées. Les valeurs possibles sont Write, Delete et Action. |


### <a name="log-analytics-workspace"></a>Espace de travail Log Analytics
La méthode héritée pour l’envoi du journal d’activité dans un espace de travail Log Analytics consiste à connecter le journal dans la configuration de l’espace de travail. 

1. À partir du menu **Espaces de travail Log Analytics** du portail Azure, sélectionnez l’espace de travail pour collecter le journal d’activité.
1. Dans la section **Sources de données de l’espace de travail** du menu de l’espace de travail, sélectionnez **Journal d’activité Azure**.
1. Cliquez sur l’abonnement que vous souhaitez connecter.

    ![Capture d'écran de l'espace de travail Log Analytics, dans lequel un journal d'activité Azure est sélectionné.](media/activity-log-collect/workspaces.png)

1. Cliquez sur **Connecter** pour connecter le journal d’activité de l’abonnement à l’espace de travail sélectionné. Si l’abonnement est déjà connecté à un autre espace de travail, cliquez d'abord sur **Déconnecter** pour le déconnecter.

    ![Connecter des espaces de travail](media/activity-log-collect/connect-workspace.png)


Pour désactiver le paramètre, effectuez la même procédure et cliquez sur **Déconnecter** pour supprimer l’abonnement de l’espace de travail.

### <a name="data-structure-changes"></a>Modifications de la structure de données
Les paramètres de diagnostic envoient les mêmes données que la méthode héritée utilisée pour envoyer le journal d’activité, avec quelques modifications de la structure de la table *AzureActivity*.

Les colonnes de la table suivante ont été dépréciées dans le schéma mis à jour. Elles existent toujours dans *AzureActivity*, mais elles ne contiennent aucune donnée. Les colonnes de remplacement ne sont pas nouvelles, mais elles contiennent les mêmes données que les colonnes dépréciées. Elles sont dans un format différent, ce qui signifie que vous devrez peut-être modifier les requêtes de journal qui les utilisent. 

| Colonne dépréciée | Colonne de remplacement |
|:---|:---|
| ActivityStatus    | ActivityStatusValue    |
| ActivitySubstatus | ActivitySubstatusValue |
| NomOpération     | OperationNameValue     |
| ResourceProvider  | ResourceProviderValue  |

> [!IMPORTANT]
> Dans certains cas, les valeurs de ces colonnes peuvent être en majuscules. Si vous avez une requête qui inclut ces colonnes, vous devez utiliser l’[opérateur =~](/azure/kusto/query/datatypes-string-operators) pour effectuer une comparaison non sensible à la casse.

La colonne suivante a été ajoutée à *AzureActivity* dans le schéma mis à jour :

- Authorization_d
- Claims_d
- Properties_d

## <a name="activity-log-analytics-monitoring-solution"></a>Solution de supervision Activity Log Analytics
La solution de supervision Azure Log Analytics sera bientôt déconseillée et remplacée par un classeur utilisant le schéma mis à jour dans l’espace de travail Log Analytics. Vous pouvez toujours utiliser la solution si vous l’avez déjà activée, mais elle ne peut être utilisée que si vous collectez le journal d’activité à l’aide des paramètres hérités. 



### <a name="use-the-solution"></a>Utiliser la solution
Les solutions de supervision sont accessibles à partir du menu **Superviser** du portail Azure. Sélectionnez **Plus** dans la section **Insights** pour ouvrir la page **Vue d’ensemble** avec les vignettes des solutions. La vignette **Azure Activity Log** affiche le nombre d'enregistrements **AzureActivity** dans votre espace de travail.

![Vignette Journaux d’activité Azure](media/collect-activity-logs/azure-activity-logs-tile.png)


Cliquez sur la vignette **Journaux d’activité Azure** pour ouvrir l'affichage **Journaux d’activité Azure**. La vue inclut les composants de visualisation dans le tableau suivant. Chaque partie répertorie jusqu'à 10 éléments répondant à ses critères en ce qui concerne l’intervalle de temps spécifié. Vous pouvez exécuter une requête de journal qui renvoie tous les enregistrements correspondants en cliquant sur **Afficher tout** en bas du composant.

![Tableau de bord Journaux d’activité Azure](media/collect-activity-logs/activity-log-dash.png)


### <a name="enable-the-solution-for-new-subscriptions"></a>Activer la solution pour les nouveaux abonnements
Vous ne pourrez bientôt plus ajouter la solution Activity Log Analytics à votre abonnement à l’aide du Portail Azure. Vous pouvez l’ajouter à l’aide de la procédure suivante avec un modèle Resource Manager. 

1. Copiez le code  JSON suivant dans un fichier appelé *ActivityLogTemplate*.json.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "defaultValue": "my-workspace",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "east us",
              "west us",
              "australia central",
              "west europe"
            ],
            "defaultValue": "australia central",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        }
      },
        "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "features": {
                    "searchVersion": 2
                }
            }
        },
        {
            "type": "Microsoft.OperationsManagement/solutions",
            "apiVersion": "2015-11-01-preview",
            "name": "[concat('AzureActivity(', parameters('workspaceName'),')')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
            ],
            "plan": {
                "name": "[concat('AzureActivity(', parameters('workspaceName'),')')]",
                "promotionCode": "",
                "product": "OMSGallery/AzureActivity",
                "publisher": "Microsoft"
            },
            "properties": {
                "workspaceResourceId": "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]",
                "containedResources": [
                    "[concat(resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName')), '/views/AzureActivity(',parameters('workspaceName'))]"
                ]
            }
        },
        {
          "type": "Microsoft.OperationalInsights/workspaces/datasources",
          "kind": "AzureActivityLog",
          "name": "[concat(parameters('workspaceName'), '/', subscription().subscriptionId)]",
          "apiVersion": "2015-11-01-preview",
          "location": "[parameters('location')]",
          "dependsOn": [
              "[parameters('WorkspaceName')]"
          ],
          "properties": {
              "linkedResourceId": "[concat(subscription().Id, '/providers/microsoft.insights/eventTypes/management')]"
          }
        }
      ]
    }    
    ```

2. Déployez le modèle à l’aide des commandes PowerShell suivantes :

    ```PowerShell
    Connect-AzAccount
    Select-AzSubscription <SubscriptionName>
    New-AzResourceGroupDeployment -Name activitysolution -ResourceGroupName <ResourceGroup> -TemplateFile <Path to template file>
    ```



## <a name="next-steps"></a>Étapes suivantes

* [Lire une vue d’ensemble des journaux de plateforme](platform-logs-overview.md)
* [Examiner le schéma d’événements du journal d’activité](activity-log-schema.md)
* [Créer un paramètre de diagnostic pour envoyer des journaux d’activité à d’autres destinations](diagnostic-settings.md)