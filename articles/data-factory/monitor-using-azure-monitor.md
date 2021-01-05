---
title: Surveiller les fabriques de données à l’aide d’Azure Monitor
description: Apprenez à utiliser Azure Monitor pour surveiller les pipelines Azure Monitor en activant les journaux de diagnostic à partir des informations de Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 35d2073dca21b4a0d48a43bed9933bb7549cf8f3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497892"
---
# <a name="monitor-and-alert-data-factory-by-using-azure-monitor"></a>Déclencher des alertes et surveiller les fabriques de données avec Azure Monitor

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Les applications cloud sont complexes, et comprennent de nombreux éléments mobiles. Les analyses fournissent des données pour vous aider à garantir que votre application reste opérationnelle et soit exécutée en toute intégrité. Les analyses vous aident également à éviter des problèmes potentiels et à résoudre les problèmes précédents. Vous pouvez utiliser les données de surveillance pour obtenir des informations détaillées sur votre application. Cette connaissance vous aide à améliorer les performances et la maintenabilité de l’application. Elle vous aide également à automatiser des actions qui nécessitent autrement une intervention manuelle.

Azure Monitor fournit des métriques et des journaux d’activité de niveau de base d’infrastructure pour la plupart des services Azure. Les journaux de diagnostic Azure sont générés par une ressource et fournissent des informations complètes et fréquentes sur le fonctionnement de cette ressource. Azure Data Factory (ADF) peut écrire les journaux de diagnostic dans Azure Monitor. Pour voir une présentation de sept minutes et la démonstration de cette fonctionnalité, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Monitor-Data-Factory-pipelines-using-Operations-Management-Suite-OMS/player]

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Monitor](../azure-monitor/overview.md).

## <a name="keeping-azure-data-factory-metrics-and-pipeline-run-data"></a>Conserver les mesures d’Azure Data Factory et les données d’exécution de pipeline

Data Factory ne stocke les données d’exécution du pipeline que pendant 45 jours. Utilisez Azure Monitor si vous souhaitez conserver ces données pendant une période plus longue. Monitor vous permet d’acheminer les journaux de diagnostic à des fins d’analyse pour multiplier les cibles.

* **Compte de stockage** : Enregistrez vos journaux de diagnostic dans un compte de stockage à des fins d’audit ou d’inspection manuelle. Vous pouvez utiliser les paramètres de diagnostic pour spécifier la durée de rétention en jours.
* **Event Hub** : Diffusez en continu les journaux vers Azure Event Hubs. Les journaux deviennent des entrées pour un service partenaire ou une solution d’analyse personnalisée telle que Power BI.
* **Log Analytics** : Analysez les journaux avec Log Analytics. L’intégration de Data Factory à Azure Monitor est utile dans les scénarios suivants :
  * Vous souhaitez écrire des requêtes complexes sur un ensemble varié de mesures publiées par la Data Factory sur Monitor. Vous pouvez créer des alertes personnalisées sur ces requêtes via Monitor.
  * Vous souhaitez surveiller l’activité de toutes les fabriques de données. Vous pouvez acheminer les données de plusieurs fabriques de données vers un seul espace de travail Monitor.

Vous pouvez également utiliser un compte de stockage ou un espace de noms Event Hub qui ne se trouve pas dans le même abonnement que la ressource émettrice des journaux. L'utilisateur qui configure le paramètre doit disposer d'un accès Azure RBAC (Contrôle d'accès en fonction du rôle Azure) approprié pour les deux abonnements.

## <a name="configure-diagnostic-settings-and-workspace"></a>Configurer les paramètres de diagnostic et l’espace de travail

Créez ou ajoutez des paramètres de diagnostic pour votre fabrique de données.

1. Dans le portail, accédez à Monitor. Sélectionnez **Paramètres** > **Paramètres de diagnostic**.

1. Sélectionnez la fabrique de données pour laquelle vous souhaitez définir un paramètre de diagnostic.

1. S’il n’existe aucun paramètre sur la fabrique de données sélectionnée, vous êtes invité à en créer un. Sélectionnez **Activer les diagnostics**.

   ![Créer un paramètre de diagnostic s’il n’en existe pas](media/data-factory-monitor-oms/monitor-oms-image1.png)

   S’il existe des paramètres sur la fabrique de données, vous voyez une liste de paramètres déjà configurés sur cette fabrique de données. Sélectionnez **Ajouter le paramètre de diagnostic**.

   ![Ajouter un paramètre de diagnostic s’il existe des paramètres](media/data-factory-monitor-oms/add-diagnostic-setting.png)

1. Nommez votre paramètre, sélectionnez **Envoyer à log Analytics**, puis sélectionnez un espace de travail dans **Espace de travail Log Analytics**.

    * En mode _Diagnostics Azure_, les journaux de diagnostic circulent dans la table _AzureDiagnostics_.

    * En mode _Spécifique à la ressource_, les journaux de diagnostic d’Azure Data Factory circulent dans les tables suivantes :
      - _ADFActivityRun_
      - _ADFPipelineRun_
      - _ADFTriggerRun_
      - _ADFSSISIntegrationRuntimeLogs_
      - _ADFSSISPackageEventMessageContext_
      - _ADFSSISPackageEventMessages_
      - _ADFSSISPackageExecutableStatistics_
      - _ADFSSISPackageExecutionComponentPhases_
      - _ADFSSISPackageExecutionDataStatistics_

      Vous pouvez sélectionner différents journaux pertinents pour vos charges de travail à envoyer vers les tableaux Log Analytics. Par exemple, si vous n’utilisez pas du tout SQL Server Integration Services (SSIS), vous n’avez pas besoin de sélectionner des journaux SSIS. Si vous souhaitez consigner les opérations de démarrage/arrêt et de maintenance du SSIS Integration Runtime (IR), vous pouvez sélectionner les journaux SSIS IR. Si vous appelez des exécutions de package SSIS via T-SQL sur SQL Server Management Studio (SSMS), SQL Server Agent ou d’autres outils désignés, vous pouvez sélectionner des journaux de packages SSIS. Si vous appelez des exécutions de package SSIS via des activités Exécuter le package SSIS dans des pipelines ADF, vous pouvez sélectionner tous les journaux.

    * Si vous sélectionnez _AllMetrics_, plusieurs métriques ADF sont mises à votre disposition pour surveiller ou déclencher des alertes, y compris les métriques pour l’activité ADF, le pipeline et les exécutions de déclencheur, ainsi que pour les opérations de runtime d’intégration de SSIS et les exécutions de packages SSIS.

   ![Nommer vos paramètres et sélectionner un espace de travail Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

    > [!NOTE]
    > Étant donné qu’une table de journal Azure ne peut pas comporter plus de 500 colonnes, nous vous **recommandons vivement** de sélectionner le _mode spécifique de la ressource_. Pour plus d’informations, consultez [Limitations connues de Log Analytics](../azure-monitor/platform/resource-logs.md#column-limit-in-azurediagnostics).

1. Sélectionnez **Enregistrer**.

Après quelques instants, le nouveau paramètre apparaît dans la liste des paramètres pour cette fabrique de données. Les journaux de diagnostic sont diffusés en continu vers cet espace de travail dès que de nouvelles données d’événement sont générées. Jusqu’à 15 minutes peuvent s’écouler entre l’émission d’un événement et son affichage dans Log Analytics.

## <a name="install-azure-data-factory-analytics-solution-from-azure-marketplace"></a>Installer la solution Azure Data Factory Analytics à partir de la Place de marché Azure

Cette solution fournit un résumé de l’intégrité globale de votre Data Factory, avec des options permettant d’examiner les détails et de résoudre les problèmes de comportements inattendus. Avec des affichages enrichis et prêts à l’emploi, vous pouvez vous familiariser avec le traitement des clés, notamment :

* Un récapitulatif des exécutions de pipeline, d’activité et de déclencheur Data Factory
* Une possibilité d’effectuer des recherches dans les exécutions d’activités Data Factory par type
* Un résumé des erreurs d’activités rencontrées et du meilleur pipeline Data Factory

1. Accédez à la **Place de marché Azure**, sélectionnez **Filtre Analytics**, puis cherchez **Azure Data Factory Analytics (préversion)**

   ![Accédez à la « Place de marché Azure », entrez « Filtre Analytics », puis sélectionnez « Azure Data Factory Analytics (préversion) »](media/data-factory-monitor-oms/monitor-oms-image3.png)

1. Détails sur **Azure Data Factory Analytics (préversion)**

   ![Détails sur « Azure Data Factory Analytics (préversion) »](media/data-factory-monitor-oms/monitor-oms-image4.png)

1. Sélectionnez **Créer**, puis créez ou sélectionnez l’**espace de travail Log Analytics**.

   ![Création d’une solution](media/data-factory-monitor-oms/monitor-log-analytics-image-5.png)

### <a name="monitor-data-factory-metrics"></a>Surveiller les métriques de Data Factory

L’installation de cette solution crée un ensemble de vues par défaut à l’intérieur de la section classeurs de l’espace de travail Log Analytics choisi. Par conséquent, les métriques suivantes sont activés :

* Exécutions d’ADF - 1) Exécutions de pipeline par Data Factory
* Exécution d’ADF - 2) Exécutions d’activités par Data Factory
* Exécutions d’ADF- 3) Exécutions de déclencheur par Data Factory
* Erreurs d’ADF - 1) 10 premières erreurs de pipeline par Data Factory
* Erreurs d’ADF - 2) 10 premières exécution d'activité par Data Factory
* Erreur d’ADF - 3) 10 premières erreurs de déclencheur par Data Factory
* Statistiques d’ADF - 1) Exécutions d’activité par type
* Statistiques d’ADF - 2) Exécutions de déclencheur par type
* Statistiques d’ADF - 3) Durée maximale d’exécutions du pipeline

![Fenêtre avec « Classeurs (préversion) » et « AzureDataFactoryAnalytics » en surbrillance](media/data-factory-monitor-oms/monitor-oms-image6.png)

Vous pouvez visualiser les métriques ci-dessus, consulter les requêtes derrière ces métriques, modifier les requêtes, créer des alertes, et exécuter d’autres action.

![Représentation graphique des exécutions de pipeline par Data Factory](media/data-factory-monitor-oms/monitor-oms-image8.png)

> [!NOTE]
> Azure Data Factory Analytics (préversion) envoie des journaux de diagnostic à des tables de destination _Spécifique de la ressource_. Vous pouvez écrire des requêtes portant sur les tables suivantes : _ADFPipelineRun_, _ADFTriggerRun_ et _ADFActivityRun_.

## <a name="data-factory-metrics"></a>Métriques de Data Factory

Monitor vous apporte une visibilité sur les performances et l’intégrité de vos charges de travail Azure. Le type de données d’analyse le plus important de Monitor est la métrique, également appelée compteur de performances. Des métriques sont émises par la plupart des ressources Azure. Monitor propose plusieurs façons de configurer et d’utiliser ces métriques à des fins d’analyse et de résolution des problèmes.

Voici quelques-unes des métriques émises par Azure Data Factory version 2 :

| **Mesure**                           | **Nom d’affichage de la métrique**                  | **Unité** | **Type d’agrégation** | **Description**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| ActivityCancelledRuns                 | Métriques d’exécutions d’activité annulées           | Count    | Total                | Nombre total d’exécutions d’activités annulées dans une fenêtre d’une minute. |
| ActivityFailedRuns                   | Métriques d’exécutions d’activité ayant échoué             | Count    | Total                | Nombre total d’exécutions d’activité ayant échoué dans une fenêtre d’une minute. |
| ActivitySucceededRuns                | Métriques d’exécutions d’activité ayant abouti          | Count    | Total                | Nombre total d’exécutions d’activité ayant réussi dans une fenêtre d’une minute. |
| PipelineCancelledRuns                 | Métriques d’exécutions de pipeline annulées           | Count    | Total                | Nombre total d’exécutions de pipelines annulées dans une fenêtre d’une minute. |
| PipelineFailedRuns                   | Métriques d’exécutions de pipeline ayant échoué             | Count    | Total                | Nombre total d’exécutions de pipeline ayant échoué dans une fenêtre d’une minute. |
| PipelineSucceededRuns                | Métriques d’exécutions de pipeline ayant abouti          | Count    | Total                | Nombre total d’exécutions de pipeline ayant réussi dans une fenêtre d’une minute. |
| TriggerCancelledRuns                  | Métriques d’exécutions de déclencheur annulées            | Count    | Total                | Nombre total d’exécutions de déclencheurs annulées dans une fenêtre d’une minute. |
| TriggerFailedRuns                    | Métriques d’exécutions de déclencheur ayant échoué              | Count    | Total                | Nombre total d’exécutions de déclencheur ayant échoué dans une fenêtre d’une minute. |
| TriggerSucceededRuns                 | Métriques d’exécutions de déclencheur ayant abouti           | Count    | Total                | Nombre total d’exécutions de déclencheur ayant réussi dans une fenêtre d’une minute. |
| SSISIntegrationRuntimeStartCancelled  | Métriques de démarrages du runtime d’intégration de SSIS annulés           | Count    | Total                | Nombre total de démarrages du runtime d’intégration de SSIS annulés dans une fenêtre d’une minute. |
| SSISIntegrationRuntimeStartFailed    | Métriques de démarrages du runtime d’intégration de SSIS ayant échoué             | Count    | Total                | Nombre total de démarrages du runtime d’intégration de SSIS ayant échoués dans une fenêtre d’une minute. |
| SSISIntegrationRuntimeStartSucceeded | Métriques de démarrages du runtime d’intégration de SSIS réussis          | Count    | Total                | Nombre total de démarrages du runtime d’intégration de SSIS ayant réussi dans une fenêtre d’une minute. |
| SSISIntegrationRuntimeStopStuck      | Métriques d’arrêts du runtime d’intégration de SSIS bloqués               | Count    | Total                | Nombre total d’arrêts du runtime d’intégration de SSIS bloqués dans une fenêtre d’une minute. |
| SSISIntegrationRuntimeStopSucceeded  | Métriques d’arrêts du runtime d’intégration de SSIS réussis           | Count    | Total                | Nombre total d’arrêts du runtime d’intégration de SSIS ayant réussi dans une fenêtre d’une minute. |
| SSISPackageExecutionCancelled         | Métriques d’exécutions du package SSIS annulées  | Count    | Total                | Nombre total d’exécutions du package SSIS annulées dans une fenêtre d’une minute. |
| SSISPackageExecutionFailed           | Métriques d’exécutions de package SSIS ayant échouées    | Count    | Total                | Nombre total d’exécutions de packages SSIS ayant échoué dans une fenêtre d’une minute. |
| SSISPackageExecutionSucceeded        | Métriques d’exécutions de package SSIS ayant réussies | Count    | Total                | Nombre total d’exécutions de packages SSIS ayant réussi dans une fenêtre d’une minute. |

Pour accéder aux métriques, suivez les instructions fournies dans [Plateforme de données Azure Monitor](../azure-monitor/platform/data-platform.md).

> [!NOTE]
> Seuls les événements d’exécution de pipeline et d’activité déclenchés et accomplis sont émis. Les exécutions en cours et de débogage **ne sont pas** émises. En revanche, les événements de **toutes** les exécutions de packages SSIS sont émis, y compris ceux qui sont terminés et en cours, indépendamment de leurs méthodes d’invocation. Par exemple, vous pouvez invoquer des exécutions de packages sur des outils SQL Server Data Tools (SSDT) compatibles avec Azure, via T-SQL sur SSMS, SQL Server Agent ou d’autres outils désignés, et en tant qu’exécutions déclenchées ou de débogage d’activités d’exécution de package SSIS dans des pipelines ADF.

## <a name="data-factory-alerts"></a>Alertes Data Factory

Connectez-vous au portail Azure, puis sélectionnez **Monitor** > **Alertes** pour créer des alertes.

![Alertes dans le menu du portail](media/monitor-using-azure-monitor/alerts_image3.png)

### <a name="create-alerts"></a>Créer des alertes

1. Sélectionnez **+ Nouvelle règle d’alerte** pour créer une nouvelle alerte.

    ![Nouvelle règle d’alerte](media/monitor-using-azure-monitor/alerts_image4.png)

1. Définissez la condition de l’alerte.

    > [!NOTE]
    > Veillez à sélectionner **Toutes** dans la liste déroulante **Filtrer par type de ressource**.

    ![« Définir la condition d’alerte » > « Sélectionner la cible », pour ouvrir le volet « Sélectionner une ressource » ](media/monitor-using-azure-monitor/alerts_image5.png)

    ![« Définir la condition d’alerte » > « Ajouter des critères », pour ouvrir le volet « Configurer la logique du signal »](media/monitor-using-azure-monitor/alerts_image6.png)

    ![Volet « Configurer la logique du signal »](media/monitor-using-azure-monitor/alerts_image7.png)

1. Définissez les détails de l’alerte.

    ![Détails de l’alerte](media/monitor-using-azure-monitor/alerts_image8.png)

1. Définissez le groupe d’actions.

    ![Créer une règle, avec « Nouveau groupe d’actions » en surbrillance](media/monitor-using-azure-monitor/alerts_image9.png)

    ![Créer un nouveau groupe d’action](media/monitor-using-azure-monitor/alerts_image10.png)

    ![Configurer le courrier, les SMS, les notifications push et la voix](media/monitor-using-azure-monitor/alerts_image11.png)

    ![Définir un groupe d’actions](media/monitor-using-azure-monitor/alerts_image12.png)

## <a name="set-up-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Configurer les journaux de diagnostic via l’API REST Azure Monitor

### <a name="diagnostic-settings"></a>Paramètres de diagnostic

Utilisez des paramètres de diagnostic pour configurer les journaux de diagnostic des ressources autres que de calcul. Les paramètres d’un contrôle de ressource présentent les caractéristiques suivantes :

* Ils spécifient l’emplacement d’envoi des journaux de diagnostic. Il peut s’agir, part exemple, d’un compte de stockage Azure, d’un Event Hub Azure ou de journaux Azure Monitor.
* Ils spécifient les catégories de journal qui sont envoyées.
* Ils spécifient la durée pendant laquelle chaque catégorie de journal doit être conservée dans un compte de stockage.
* Une durée de rétention de zéro jour signifie que les journaux d’activité sont conservés indéfiniment. Autrement, la valeur peut également être n’importe quel nombre de jours, compris entre 1 et 2 147 483 647.
* Si des stratégies de rétention sont définies, mais que le stockage des journaux d’activité dans un compte de stockage est désactivé, les stratégies de rétention n’ont aucun effet. Par exemple, cette condition peut se produire lorsque seules les options Event Hubs ou journaux Azure Monitor sont sélectionnées.
* Les stratégies de rétention sont appliquées par jour. La séparation entre les jours se produit à minuit en temps universel coordonné (UTC). À la fin de la journée, les journaux des jours qui dépassent la stratégie de rétention sont supprimés. Par exemple, si vous avez une stratégie de rétention d’une journée, au début de la journée les journaux antérieurs à la veille sont supprimés.

### <a name="enable-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Activer les journaux de diagnostic via l’API REST Azure Monitor

#### <a name="create-or-update-a-diagnostics-setting-in-the-monitor-rest-api"></a>Créer ou mettre à jour un paramètre de diagnostic dans l’API REST Azure Monitor

##### <a name="request"></a>Requête

```
PUT
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>headers

* Remplacez `{api-version}` par `2016-09-01`.
* Remplacez `{resource-id}` par l’ID de la ressource dont vous voulez modifier les paramètres de diagnostic. Pour plus d'informations, consultez la rubrique [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* Attribuez à l’en-tête `Content-Type` la valeur `application/json`.
* Définissez l’en-tête d’autorisation sur le jeton web JSON que vous avez obtenu d’Azure Active Directory (AAD). Pour plus d’informations, consultez [Authentification des demandes](../active-directory/develop/authentication-vs-authorization.md).

##### <a name="body"></a>body

```json
{
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "metrics": [
        ],
        "logs": [
                {
                    "category": "PipelineRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "TriggerRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "ActivityRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                }
            ]
    },
    "location": ""
}
```

| Propriété | Type | Description |
| --- | --- | --- |
| **storageAccountId** |String | ID de ressource du compte de stockage auquel vous voulez envoyer les journaux de diagnostic. |
| **serviceBusRuleId** |String | ID de règle Service Bus de l’espace de noms Service Bus dans lequel vous voulez que les Event Hubs soient créés pour la diffusion en continu des journaux de diagnostic. L’ID de règle a le format `{service bus resource ID}/authorizationrules/{key name}`.|
| **workspaceId** | Type complexe | Tableau de fragments de temps de métrique et leurs stratégies de rétention. La valeur de cette propriété est vide. |
|**metrics**| Valeurs de paramètre de l’exécution de pipeline à passer au pipeline appelé| Objet JSON qui mappe des noms de paramètres à des valeurs d’arguments. |
| **logs**| Type complexe| Nom d’une catégorie de journal de diagnostic pour un type de ressource. Pour obtenir la liste des catégories de journal de diagnostic pour une ressource, effectuez une opération d’obtention (GET) des paramètres de diagnostic. |
| **category**| String| Tableau de catégories de journal et leurs stratégies de rétention. |
| **timeGrain** | String | Granularité des métriques capturées au format de durée ISO 8601. La valeur de la propriété doit être `PT1M`, qui spécifie une minute. |
| **activé**| Boolean | Indique si la collection de la métrique ou catégorie de journal est activée pour cette ressource. |
| **retentionPolicy**| Type complexe| Décrit la stratégie de rétention pour une métrique ou une catégorie de journal. Cette propriété est utilisée uniquement pour les comptes de stockage. |
|**days**| Int| Nombre de jours durant lesquels les métriques ou journaux sont conservés. Si la valeur de la propriété est 0, les journaux sont conservés indéfiniment. Cette propriété est utilisée uniquement pour les comptes de stockage. |

##### <a name="response"></a>response

200 OK.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```

#### <a name="get-information-about-diagnostics-settings-in-the-monitor-rest-api"></a>Obtenez des informations sur le paramètre de diagnostic dans l’API REST Azure Monitor

##### <a name="request"></a>Requête

```
GET
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>headers

* Remplacez `{api-version}` par `2016-09-01`.
* Remplacez `{resource-id}` par l’ID de la ressource dont vous voulez modifier les paramètres de diagnostic. Pour plus d'informations, consultez la rubrique [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* Attribuez à l’en-tête `Content-Type` la valeur `application/json`.
* Définissez l’en-tête d’autorisation sur un jeton web JSON que vous avez obtenu d’Azure Active Directory. Pour plus d’informations, consultez [Authentification des demandes](../active-directory/develop/authentication-vs-authorization.md).

##### <a name="response"></a>response

200 OK.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.Storage/storageAccounts/azmonlogs",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.EventHub/namespaces/shloeventhub/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/ADF/providers/Microsoft.OperationalInsights/workspaces/mihaipie",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```
Pour plus d’informations, voir [Paramètres de diagnostic](/rest/api/monitor/diagnosticsettings).

## <a name="schema-of-logs-and-events"></a>Schéma des journaux et des événements

### <a name="monitor-schema"></a>Schéma Azure Monitor

#### <a name="activity-run-log-attributes"></a>Attributs du journal d’exécution d’activité

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "activityRunId":"",
   "pipelineRunId":"",
   "resourceId":"",
   "category":"ActivityRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "activityName":"",
   "start":"",
   "end":"",
   "properties":
       {
          "Input": "{
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "BlobSink"
              }
           }",
          "Output": "{"dataRead":121,"dataWritten":121,"copyDuration":5,
               "throughput":0.0236328132,"errors":[]}",
          "Error": "{
              "errorCode": "null",
              "message": "null",
              "failureType": "null",
              "target": "CopyBlobtoBlob"
        }
    }
}
```

| Propriété | Type | Description | Exemple |
| --- | --- | --- | --- |
| **Niveau** |String | Niveau des journaux de diagnostic. Pour les journaux d’exécution d’activité, affectez la valeur 4 à la propriété. | `4` |
| **correlationId** |String | ID unique pour le suivi d’une demande particulière. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | Heure de l’événement au format UTC d’intervalle de temps `YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**activityRunId**| String| ID de l’exécution d’activité. | `3a171e1f-b36e-4b80-8a54-5625394f4354` |
|**pipelineRunId**| String| ID de l’exécution de pipeline. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| String | ID associé à la ressource de fabrique de données. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | Catégorie des journaux de diagnostic. Affectez la valeur `ActivityRuns` à la propriété. | `ActivityRuns` |
|**level**| String | Niveau des journaux de diagnostic. Affectez la valeur `Informational` à la propriété. | `Informational` |
|**operationName**| String | Nom de l’activité avec cet état. Si l’activité est la pulsation de début, la valeur de la propriété est `MyActivity -`. Si l’activité est la pulsation de fin, la valeur de la propriété est `MyActivity - Succeeded`. | `MyActivity - Succeeded` |
|**pipelineName**| String | Nom du pipeline. | `MyPipeline` |
|**activityName**| String | Nom de l’activité. | `MyActivity` |
|**start**| String | Heure de début des exécutions d’activité au format UTC d’intervalle de temps. | `2017-06-26T20:55:29.5007959Z`|
|**end**| String | Heure de fin des exécutions d’activité au format UTC d’intervalle de temps. Si le journal de diagnostic indique qu’une activité a démarré mais qu’elle n’est pas encore terminée, la valeur de la propriété est `1601-01-01T00:00:00Z`. | `2017-06-26T20:55:29.5007959Z` |

#### <a name="pipeline-run-log-attributes"></a>Attributs du journal d’exécution de pipeline

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "runId":"",
   "resourceId":"",
   "category":"PipelineRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "start":"",
   "end":"",
   "status":"",
   "properties":
    {
      "Parameters": {
        "<parameter1Name>": "<parameter1Value>"
      },
      "SystemParameters": {
        "ExecutionStart": "",
        "TriggerId": "",
        "SubscriptionId": ""
      }
    }
}
```

| Propriété | Type | Description | Exemple |
| --- | --- | --- | --- |
| **Niveau** |String | Niveau des journaux de diagnostic. Pour les journaux d’exécution d’activité, affectez la valeur 4 à la propriété. | `4` |
| **correlationId** |String | ID unique pour le suivi d’une demande particulière. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | Heure de l’événement au format UTC d’intervalle de temps `YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**runId**| String| ID de l’exécution de pipeline. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| String | ID associé à la ressource de fabrique de données. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | Catégorie des journaux de diagnostic. Affectez la valeur `PipelineRuns` à la propriété. | `PipelineRuns` |
|**level**| String | Niveau des journaux de diagnostic. Affectez la valeur `Informational` à la propriété. | `Informational` |
|**operationName**| String | Nom du pipeline, ainsi que son état. Une fois l’exécution du pipeline terminée, la valeur de la propriété est `Pipeline - Succeeded`. | `MyPipeline - Succeeded`. |
|**pipelineName**| String | Nom du pipeline. | `MyPipeline` |
|**start**| String | Heure de début des exécutions d’activité au format UTC d’intervalle de temps. | `2017-06-26T20:55:29.5007959Z`. |
|**end**| String | Heure de fin des exécutions d’activité au format UTC d’intervalle de temps. Si le journal de diagnostic indique qu’une activité a démarré mais qu’elle n’est pas encore terminée, la valeur de la propriété est `1601-01-01T00:00:00Z`.  | `2017-06-26T20:55:29.5007959Z` |
|**statut**| String | État final de l’exécution de pipeline. Les valeurs possibles de la propriété sont `Succeeded` et `Failed`. | `Succeeded`|

#### <a name="trigger-run-log-attributes"></a>Attributs du journal d’exécution de déclencheur

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "triggerId":"",
   "resourceId":"",
   "category":"TriggerRuns",
   "level":"Informational",
   "operationName":"",
   "triggerName":"",
   "triggerType":"",
   "triggerEvent":"",
   "start":"",
   "status":"",
   "properties":
   {
      "Parameters": {
        "TriggerTime": "",
       "ScheduleTime": ""
      },
      "SystemParameters": {}
    }
}
```

| Propriété | Type | Description | Exemple |
| --- | --- | --- | --- |
| **Niveau** |String | Niveau des journaux de diagnostic. Pour les journaux d’exécution d’activité, affectez la valeur 4 à la propriété. | `4` |
| **correlationId** |String | ID unique pour le suivi d’une demande particulière. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | String | Heure de l’événement au format UTC d’intervalle de temps `YYYY-MM-DDTHH:MM:SS.00000Z`. | `2017-06-28T21:00:27.3534352Z` |
|**triggerId**| String| ID de l’exécution du déclencheur. | `08587023010602533858661257311` |
|**resourceId**| String | ID associé à la ressource de fabrique de données. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| String | Catégorie des journaux de diagnostic. Affectez la valeur `PipelineRuns` à la propriété. | `PipelineRuns` |
|**level**| String | Niveau des journaux de diagnostic. Affectez la valeur `Informational` à la propriété. | `Informational` |
|**operationName**| String | Nom du déclencheur avec son état final, qui indique si le déclencheur a fonctionné correctement. Si la pulsation a réussi, la valeur de la propriété est `MyTrigger - Succeeded`. | `MyTrigger - Succeeded` |
|**triggerName**| String | Nom du déclencheur. | `MyTrigger` |
|**triggerType**| String | Type du déclencheur. Les valeurs possibles de la propriété sont `Manual Trigger` et `Schedule Trigger`. | `ScheduleTrigger` |
|**triggerEvent**| String | Événement du déclencheur. | `ScheduleTime - 2017-07-06T01:50:25Z` |
|**start**| String | Heure de début de l’action du défenseur au format UTC d’intervalle de temps. | `2017-06-26T20:55:29.5007959Z`|
|**statut**| String | État final indiquant si le déclencheur a fonctionné correctement. Les valeurs possibles de la propriété sont `Succeeded` et `Failed`. | `Succeeded`|

#### <a name="ssis-integration-runtime-log-attributes"></a>Attributs du journal du runtime d’intégration SSIS

Voici les attributs du journal des opérations de démarrage/arrêt/maintenance du runtime d’intégration de SSIS.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "resultType": "",
   "properties": {
      "message": ""
   },
   "resourceId": ""
}
```

| Propriété                   | Type   | Description                                                   | Exemple                        |
| -------------------------- | ------ | ------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Nom de votre opération IR SSIS                            | `Start/Stop/Maintenance` |
| **category**               | String | Catégorie de journaux de diagnostic                               | `SSISIntegrationRuntimeLogs` |
| **correlationId**          | String | ID unique pour le suivi d’une opération particulière             | `f13b159b-515f-4885-9dfa-a664e949f785Deprovision0059035558` |
| **dataFactoryName**        | String | Nom de votre ADF                                          | `MyADFv2` |
| **integrationRuntimeName** | String | Nom de votre IR SSIS                                      | `MySSISIR` |
| **level**                  | String | Niveau des journaux de diagnostic                                  | `Informational` |
| **resultType**             | String | Résultat de votre opération IR SSIS                          | `Started/InProgress/Succeeded/Failed` |
| **message**                | String | Message de sortie de votre opération IR SSIS                  | `The stopping of your SSIS integration runtime has succeeded.` |
| **resourceId**             | String | ID unique de la ressource ADF                            | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-message-context-log-attributes"></a>Attributs du journal de contexte des messages d’événements SSIS

Voici les attributs de journal des conditions liées aux messages d’événement générés par des exécutions de package SSIS sur votre runtime d’intégration de SSIS. Ils transmettent des informations comme [la table ou la vue de contexte de message d’événement du catalogue SSIS (SSISDB)](/sql/integration-services/system-views/catalog-event-message-context?view=sql-server-ver15) qui affiche des valeurs d’exécution de nombreuses propriétés du package SSIS. Ils sont générés lorsque vous sélectionnez le niveau de journalisation `Basic/Verbose` qui est utile pour la vérification de la conformité/du débogage.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "contextDepth": "",
      "packagePath": "",
      "contextType": "",
      "contextSourceName": "",
      "contextSourceId": "",
      "propertyName": "",
      "propertyValue": ""
   },
   "resourceId": ""
}
```

| Propriété                   | Type   | Description                                                          | Exemple                        |
| -------------------------- | ------ | -------------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z`        | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Ce paramètre est défini sur `YourSSISIRName-SSISPackageEventMessageContext`       | `mysqlmissisir-SSISPackageEventMessageContext` |
| **category**               | String | Catégorie de journaux de diagnostic                                      | `SSISPackageEventMessageContext` |
| **correlationId**          | String | ID unique pour le suivi d’une opération particulière                    | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nom de votre ADF                                                 | `MyADFv2` |
| **integrationRuntimeName** | String | Nom de votre IR SSIS                                             | `MySSISIR` |
| **level**                  | String | Niveau des journaux de diagnostic                                         | `Informational` |
| **operationId**            | String | ID unique pour le suivi d’une opération particulière dans SSISDB          | `1` (1 désigne les opérations liées aux packages **non** stockées dans SSISDB/invoquées via T-SQL) |
| **contextDepth**           | String | Profondeur du contexte de votre message d’événement                              | `0` (0 désigne le contexte avant le démarrage de l’exécution du package, 1 signifie le contexte lorsqu’une erreur se produit et qu’elle augmente à mesure que le contexte est plus éloigné de l’erreur) |
| **packagePath**            | String | Chemin d’accès de l’objet du package en tant que source de contexte de message d’événement      | `\Package` |
| **contextType**            | String | Type d’accès de l’objet du package en tant que source de contexte de message d’événement      | `60`(voir [plus de types de contexte](/sql/integration-services/system-views/catalog-event-message-context?view=sql-server-ver15#remarks)) |
| **contextSourceName**      | String | Nom d’accès de l’objet du package en tant que source de contexte de message d’événement      | `MyPackage` |
| **contextSourceId**        | String | ID unique d’accès de l’objet du package en tant que source de contexte de message d’événement | `{E2CF27FB-EA48-41E9-AF6F-3FE938B4ADE1}` |
| **propertyName**           | String | Nom de la propriété de l’objet du package en tant que source de contexte de message d’événement   | `DelayValidation` |
| **propertyValue**          | String | Valeur de la propriété de l’objet du package en tant que source de contexte de message d’événement  | `False` |
| **resourceId**             | String | ID unique de la ressource ADF                                   | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-messages-log-attributes"></a>Attributs du journal des messages d’événements SSIS

Voici les attributs de journal des messages d’événement générés par les exécutions de package SSIS sur votre runtime d’intégration SSIS. Ils transmettent des informations comme [la table ou la vue des messages d’événements SSISDB](/sql/integration-services/system-views/catalog-event-messages?view=sql-server-ver15) qui affichent le texte ou les métadonnées détaillés des messages d’événement. Ils sont générés à n’importe quel niveau de journalisation, sauf `None`.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "messageTime": "",
      "messageType": "",
      "messageSourceType": "",
      "message": "",
      "packageName": "",
      "eventName": "",
      "messageSourceName": "",
      "messageSourceId": "",
      "subcomponentName": "",
      "packagePath": "",
      "executionPath": "",
      "threadId": ""
   }
}
```

| Propriété                   | Type   | Description                                                        | Exemple                        |
| -------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                   | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Ce paramètre est défini sur `YourSSISIRName-SSISPackageEventMessages`           | `mysqlmissisir-SSISPackageEventMessages` |
| **category**               | String | Catégorie de journaux de diagnostic                                    | `SSISPackageEventMessages` |
| **correlationId**          | String | ID unique pour le suivi d’une opération particulière                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nom de votre ADF                                               | `MyADFv2` |
| **integrationRuntimeName** | String | Nom de votre IR SSIS                                           | `MySSISIR` |
| **level**                  | String | Niveau des journaux de diagnostic                                       | `Informational` |
| **operationId**            | String | ID unique pour le suivi d’une opération particulière dans SSISDB        | `1` (1 désigne les opérations liées aux packages **non** stockées dans SSISDB/invoquées via T-SQL) |
| **messageTime**            | String | Heure de création du message d’événement au format UTC          | `2017-06-28T21:00:27.3534352Z` |
| **messageType**            | String | Type du message d'événement                                     | `70`(voir [plus de types de messages](/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database?view=sql-server-ver15#remarks)) |
| **messageSourceType**      | String | Type de source du message d'événement                              | `20`(voir [plus de types de sources de messages](/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database?view=sql-server-ver15#remarks)) |
| **message**                | String | Texte du message d'événement                                     | `MyPackage:Validation has started.` |
| **packageName**            | String | Nom de votre fichier de package exécuté                             | `MyPackage.dtsx` |
| **eventName**              | String | Nom de l’événement au moment de l’exécution                                 | `OnPreValidate` |
| **messageSourceName**      | String | Nom d’accès du composant du package en tant que source du message d’événement         | `Data Flow Task` |
| **messageSourceId**        | String | ID unique d’accès du composant du package en tant que source du message d’événement    | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |
| **subcomponentName**       | String | Nom d’accès du composant de flux de données en tant que source du message d’événement       | `SSIS.Pipeline` |
| **packagePath**            | String | Chemin d’accès de l’objet du package en tant que source du message d’événement            | `\Package\Data Flow Task` |
| **executionPath**          | String | Chemin d’accès complet du package parent au composant exécuté            | `\Transformation\Data Flow Task` (Ce chemin d'accès capture également les itérations d'un composant) |
| **threadId**               | String | ID unique du thread exécuté lorsque votre message d’événement est journalisé | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |

#### <a name="ssis-executable-statistics-log-attributes"></a>Attributs du journal des statistiques exécutables SSIS

Voici les attributs de journal des statistiques exécutables générées par les exécutions de package SSIS sur votre runtime d’intégration SSIS, où les exécutables sont des conteneurs ou des tâches dans les flux de contrôle des packages. Ils transmettent des informations comme [la table ou la vue des statistiques exécutables SSISDB](/sql/integration-services/system-views/catalog-executable-statistics?view=sql-server-ver15) qui affiche une ligne pour chaque exécutable en cours d’exécution, y compris ses itérations. Ils sont générés à n’importe quel niveau de journalisation, sauf `None` et utiles pour identifier les goulots d’étranglement/défaillances au niveau des tâches.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "executionPath": "",
      "startTime": "",
      "endTime": "",
      "executionDuration": "",
      "executionResult": "",
      "executionValue": ""
   },
   "resourceId": ""
}
```

| Propriété                   | Type   | Description                                                      | Exemple                        |
| -------------------------- | ------ | ---------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z`    | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Ce paramètre est défini sur `YourSSISIRName-SSISPackageExecutableStatistics`  | `mysqlmissisir-SSISPackageExecutableStatistics` |
| **category**               | String | Catégorie de journaux de diagnostic                                  | `SSISPackageExecutableStatistics` |
| **correlationId**          | String | ID unique pour le suivi d’une opération particulière                | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nom de votre ADF                                             | `MyADFv2` |
| **integrationRuntimeName** | String | Nom de votre IR SSIS                                         | `MySSISIR` |
| **level**                  | String | Niveau des journaux de diagnostic                                     | `Informational` |
| **executionId**            | String | ID unique pour le suivi d’une exécution particulière dans SSISDB      | `1` (1 désigne les exécutions liées aux packages **non** stockées dans SSISDB/invoquées via T-SQL) |
| **executionPath**          | String | Chemin d’accès complet du package parent au composant exécuté          | `\Transformation\Data Flow Task` (Ce chemin d'accès capture également les itérations d'un composant) |
| **startTime**              | String | Heure à laquelle le fichier exécutable entre dans la phase de pré-exécution au format UTC  | `2017-06-28T21:00:27.3534352Z` |
| **endTime**                | String | Heure à laquelle le fichier exécutable entre dans la phase de post-exécution au format UTC | `2017-06-28T21:00:27.3534352Z` |
| **executionDuration**      | String | Heure d’exécution de l’exécutable en millisecondes                   | `1,125` |
| **executionResult**        | String | Résultat de l’exécution de l’exécutable                                 | `0` (0 signifie réussite, 1 signifie échec, 2 signifie fin et 3 signifie annulation) |
| **executionValue**         | String | Valeur définie par l’utilisateur retournée par l’exécutable en cours d’exécution            | `1` |
| **resourceId**             | String | ID unique de la ressource ADF                               | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-component-phases-log-attributes"></a>Attributs du journal des phases du composant d’exécution SSIS

Voici les attributs de journal des statistiques de runtime pour les composants de flux de données générés par des exécutions de package SSIS sur votre runtime d’intégration SSIS. Ils transmettent des informations comme [la table ou la vue des phases du composant d’exécution SSISDB](/sql/integration-services/system-views/catalog-execution-component-phases?view=sql-server-ver15) qui montrent le temps passé par les composants de flux de données dans toutes leurs phases d’exécution. Ils sont générés lorsque vous sélectionnez le niveau de journalisation `Performance/Verbose` qui est utile pour la capture des statistiques d’exécution du flux de données.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "subcomponentName": "",
      "phase": "",
      "startTime": "",
      "endTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Propriété                   | Type   | Description                                                         | Exemple                        |
| -------------------------- | ------ | ------------------------------------------------------------------- | ------------------------------ |
| **time**                   | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z`       | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | String | Ce paramètre est défini sur `YourSSISIRName-SSISPackageExecutionComponentPhases` | `mysqlmissisir-SSISPackageExecutionComponentPhases` |
| **category**               | String | Catégorie de journaux de diagnostic                                     | `SSISPackageExecutionComponentPhases` |
| **correlationId**          | String | ID unique pour le suivi d’une opération particulière                   | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | String | Nom de votre ADF                                                | `MyADFv2` |
| **integrationRuntimeName** | String | Nom de votre IR SSIS                                            | `MySSISIR` |
| **level**                  | String | Niveau des journaux de diagnostic                                        | `Informational` |
| **executionId**            | String | ID unique pour le suivi d’une exécution particulière dans SSISDB         | `1` (1 désigne les exécutions liées aux packages **non** stockées dans SSISDB/invoquées via T-SQL) |
| **packageName**            | String | Nom de votre fichier de package exécuté                              | `MyPackage.dtsx` |
| **taskName**               | String | Nom de la tâche de flux de données exécutée                                 | `Data Flow Task` |
| **subcomponentName**       | String | Nom du composant de flux de données                                     | `Derived Column` |
| **phase**                  | String | Nom de la phase d'exécution                                         | `AcquireConnections` |
| **startTime**              | String | Heure de début de la phase d’exécution au format UTC                  | `2017-06-28T21:00:27.3534352Z` |
| **endTime**                | String | Heure de fin de la phase d’exécution au format UTC                    | `2017-06-28T21:00:27.3534352Z` |
| **executionPath**          | String | Chemin d'exécution de la tâche de flux de données                            | `\Transformation\Data Flow Task` |
| **resourceId**             | String | ID unique de la ressource ADF                                  | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-data-statistics-log-attributes"></a>Attributs du journal des statistiques des données d’exécution SSIS

Voici les attributs de journal des déplacements de données via chaque tronçon de pipelines de flux de données, de l’amont vers les composants en aval, générés par les exécutions de package SSIS sur votre IR SSIS. Ils transmettent des informations comme [la table ou la vue des statistiques de données d’exécution SSISDB](/sql/integration-services/system-views/catalog-execution-data-statistics?view=sql-server-ver15) qui affiche le nombre de lignes de données déplacées via des tâches de flux de données. Ils sont générés lorsque vous sélectionnez le niveau de journalisation `Verbose` qui est utile pour calculer le débit du flux de données.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "dataflowPathIdString": "",
      "dataflowPathName": "",
      "sourceComponentName": "",
      "destinationComponentName": "",
      "rowsSent": "",
      "createdTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Propriété                     | Type   | Description                                                        | Exemple                        |
| ---------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                     | String | Heure de l’événement au format UTC :`YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**            | String | Ce paramètre est défini sur `YourSSISIRName-SSISPackageExecutionDataStatistics` | `mysqlmissisir-SSISPackageExecutionDataStatistics` |
| **category**                 | String | Catégorie de journaux de diagnostic                                    | `SSISPackageExecutionDataStatistics` |
| **correlationId**            | String | ID unique pour le suivi d’une opération particulière                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**          | String | Nom de votre ADF                                               | `MyADFv2` |
| **integrationRuntimeName**   | String | Nom de votre IR SSIS                                           | `MySSISIR` |
| **level**                    | String | Niveau des journaux de diagnostic                                       | `Informational` |
| **executionId**              | String | ID unique pour le suivi d’une exécution particulière dans SSISDB        | `1` (1 désigne les exécutions liées aux packages **non** stockées dans SSISDB/invoquées via T-SQL) |
| **packageName**              | String | Nom de votre fichier de package exécuté                             | `MyPackage.dtsx` |
| **taskName**                 | String | Nom de la tâche de flux de données exécutée                                | `Data Flow Task` |
| **dataflowPathIdString**     | String | ID unique du chemin d’accès du flux de données de suivi                          | `Paths[SQLDB Table3.ADO NET Source Output]` |
| **dataflowPathName**         | String | Nom du chemin d'accès de flux de données                                         | `ADO NET Source Output` |
| **sourceComponentName**      | String | Nom du composant de flux de données qui envoie les données                    | `SQLDB Table3` |
| **destinationComponentName** | String | Nom du composant de flux de données qui reçoit les données                 | `Derived Column` |
| **rowsSent**                 | String | Nombre de lignes envoyées par composant source                        | `500` |
| **createdTime**              | String | Heure à laquelle les valeurs de ligne sont obtenues au format UTC                | `2017-06-28T21:00:27.3534352Z` |
| **executionPath**            | String | Chemin d'exécution de la tâche de flux de données                           | `\Transformation\Data Flow Task` |
| **resourceId**               | String | ID unique de la ressource ADF                                 | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

### <a name="log-analytics-schema"></a>Schéma Log Analytics

Log Analytics hérite du schéma d’Azure Monitor avec les exceptions suivantes :

* La première lettre de chaque nom de colonne est majuscule. Par exemple, le nom de colonne « correlationId » dans Monitor est « CorrelationId » dans Log Analytics.
* Il n’y a pas de colonne « Level ».
* La colonne des « properties » des propriétés dynamiques est conservée en tant que type d’objet blob JSON dynamique.

    | Colonne Azure Monitor | Colonne Log Analytics | Type |
    | --- | --- | --- |
    | $.properties.UserProperties | UserProperties | Dynamique |
    | $.properties.Annotations | Annotations | Dynamique |
    | $.properties.Input | Entrée | Dynamique |
    | $.properties.Output | Output | Dynamique |
    | $.properties.Error.errorCode | ErrorCode | int |
    | $.properties.Error.message | ErrorMessage | string |
    | $.properties.Error | Error | Dynamique |
    | $.properties.Predecessors | Predecessors | Dynamique |
    | $.properties.Parameters | Paramètres | Dynamique |
    | $.properties.SystemParameters | SystemParameters | Dynamique |
    | $.properties.Tags | Balises | Dynamique |

## <a name="monitor-ssis-operations-with-azure-monitor"></a>Surveiller les opérations SSIS avec Azure Monitor

Pour lever & déplacer vos charges de travail SSIS, vous pouvez [approvisionner le runtime d’intégration SSIS dans ADF](./tutorial-deploy-ssis-packages-azure.md) qui prend en charge les aspects suivants :

- L’exécution de packages déployés dans le catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance (modèle de déploiement de projet)
- L’exécution de packages déployés dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package)

Une fois approvisionnée, vous pouvez [vérifier l’état opérationnel de l’IR SSIS à l’aide de Azure PowerShell ou le **surveiller** sur le hub du portail ADF](./monitor-integration-runtime.md#azure-ssis-integration-runtime). Avec le modèle de déploiement de projets, les journaux d’exécution de packages SSIS sont stockés dans des tables ou vues internes SSISDB, de sorte que vous pouvez les interroger, analyser et présenter visuellement à l’aide d’outils désignés tels que SSMS. Avec le modèle de déploiement de package, les journaux d’exécution de packages SSIS peuvent être stockés dans un système de fichiers ou Azure Files en tant que fichiers CSV que vous devez toujours analyser et traiter à l’aide d’autres outils désignés avant de pouvoir les interroger, analyser et présenter visuellement.

Désormais, avec l’intégration d’[Azure Monitor ](../azure-monitor/platform/data-platform.md), vous pouvez interroger, analyser et présenter visuellement l’ensemble des métriques et journaux générés à partir d’opérations de runtime d’intégration SSIS et d’exécutions de packages SSIS sur portail Azure. Vous pouvez en outre déclencher des alertes sur celles-ci.

### <a name="configure-diagnostic-settings-and-workspace-for-ssis-operations"></a>Configurer les paramètres de diagnostic et l’espace de travail pour les opérations SSIS

Pour envoyer toutes les métriques et tous les journaux générés à partir des opérations IR SSIS et des exécutions de packages SSIS à Azure Monitor, vous devez [configurer des paramètres de diagnostic et un espace de travail pour votre ADF](#configure-diagnostic-settings-and-workspace).

### <a name="ssis-operational-metrics"></a>Métriques opérationnelles SSIS

Les [métriques](../azure-monitor/platform/data-platform-metrics.md) opérationnelles SSIS sont des compteurs de performances ou des valeurs numériques décrivant l’état des opérations de démarrage et arrêt du runtime d’intégration SSIS, ainsi que des exécutions de packages SSIS à un moment donné. Elles font partie des [métriques d’ADF dans Azure Monitor](#data-factory-metrics).

Quand vous configurez les paramètres de diagnostic et l’espace de travail de votre ADF sur Azure Monitor, le fait de cocher la case _AllMetrics_ rend les métriques opérationnelles SSIS disponibles pour [l’analyse interactive à l’aide d’Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md), [la présentation sur le tableau de bord Azure](../azure-monitor/learn/tutorial-app-dashboards.md)et [les alertes en quasi-temps réel](../azure-monitor/platform/alerts-metric.md).

![Nommer vos paramètres et sélectionner un espace de travail Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

### <a name="ssis-operational-alerts"></a>Alertes opérationnelles SSIS

Pour déclencher des alertes sur les métriques opérationnelles SSIS à partir du portail ADF, [sélectionnez la page **Alertes et métriques** du hub ADF **Monitor** et suivez les instructions fournies](./monitor-visually.md#alerts).

![Multiplier les alertes opérationnelles SSIS à partir du portail ADF](media/data-factory-monitor-oms/data-factory-monitor-alerts-ssis.png)

Pour déclencher des alertes sur les métriques opérationnelles SSIS à partir du Portail Azure, [sélectionnez la page **Alertes** du hub Azure **Monitor** et suivez les instructions fournies](#data-factory-alerts).

![Multiplier les alertes opérationnelles SSIS à partir du Portail Azure](media/data-factory-monitor-oms/azure-monitor-alerts-ssis.png)

### <a name="ssis-operational-logs"></a>Journaux opérationnels SSIS

Les [journaux](../azure-monitor/platform/data-platform-logs.md) opérationnels SSIS sont des événements générés par les opérations IR SSIS et les exécutions de packages SSIS qui fournissent suffisamment de contexte sur tous les problèmes identifiés et sont utiles pour l’analyse de la cause racine. 

Quand vous configurez les paramètres de diagnostic et l’espace de travail pour votre ADF sur Azure Monitor, vous pouvez sélectionner les journaux des opérations SSIS appropriés et les envoyer à Log Analytics basé sur Azure Data Explorer. À partir de là, ils seront mis à disposition pour [l’analyse à l’aide d’un langage de requête riche](../azure-monitor/log-query/log-query-overview.md), [la présentation sur le tableau de bord Azure](../azure-monitor/learn/tutorial-app-dashboards.md)et [les alertes en quasi-temps réel](../azure-monitor/platform/alerts-log.md).

![Nommer vos paramètres et sélectionner un espace de travail Log Analytics](media/data-factory-monitor-oms/monitor-oms-image2.png)

Les schémas et le contenu des journaux d’exécution de packages SSIS dans Azure Monitor et Log Analytics sont similaires à ceux des tables ou vues internes de SSISDB.

| Catégories de journaux Azure Monitor          | Tableaux Log Analytics                     | Tableaux/vues internes SSISDB              |
| ------------------------------------- | ---------------------------------------- | ----------------------------------------- |
| `SSISIntegrationRuntimeLogs`          | `ADFSSISIntegrationRuntimeLogs`          |                                           |
| `SSISPackageEventMessageContext`      | `ADFSSISPackageEventMessageContext`      | `[internal].[event_message_context]`      |
| `SSISPackageEventMessages`            | `ADFSSISPackageEventMessages`            | `[internal].[event_messages]`             |
| `SSISPackageExecutableStatistics`     | `ADFSSISPackageExecutableStatistics`     | `[internal].[executable_statistics]`      |
| `SSISPackageExecutionComponentPhases` | `ADFSSISPackageExecutionComponentPhases` | `[internal].[execution_component_phases]` |
| `SSISPackageExecutionDataStatistics`  | `ADFSSISPackageExecutionDataStatistics`  | `[internal].[execution_data_statistics]`  |

Pour plus d’informations sur les attributs/propriétés du journal opérationnel SSIS, consultez [Schémas sur Azure Monitor et Log Analytics pour ADF](#schema-of-logs-and-events).

Les journaux d’exécution de package SSIS sélectionnés sont toujours envoyés à Log Analytics, indépendamment de leurs méthodes d’invocation. Par exemple, vous pouvez appeler des exécutions de packages sur des SSDT compatible avec Azure, via T-SQL sur SSMS, SQL Server Agent ou d’autres outils désignés, et comme des exécutions déclenchées ou de débogage d’activités d’exécution de package SSIS dans des pipelines ADF.

Lorsque vous interrogez des journaux des opérations de runtime d’intégration SSIS sur Logs Analytics, vous pouvez utiliser les propriétés **OperationName** et **ResultType** définies respectivement sur `Start/Stop/Maintenance` et `Started/InProgress/Succeeded/Failed`. 

![Interrogation des journaux des opérations de runtime d’intégration SSIS sur Log Analytics](media/data-factory-monitor-oms/log-analytics-query.png)

Lors de l’interrogation des journaux d’exécution des packages SSIS sur Logs Analytics, vous pouvez les joindre à l’aide des propriétés **OperationId**/**ExecutionId**/**CorrelationId**. Les propriétés **OperationId**/**ExecutionId** ont toujours la valeur `1` pour toutes les opérations/exécutions relatives à des packages **non** stockés dans SSISDB/invoqués via T-SQL.

![Interroger les journaux d’exécution des packages SSIS sur Log Analytics](media/data-factory-monitor-oms/log-analytics-query2.png)

## <a name="next-steps"></a>Étapes suivantes
[Surveiller et gérer les pipelines par programmation](monitor-programmatically.md)