---
title: Création d’alertes de métrique de journaux d’activité dans Azure Monitor
description: Didacticiel sur la création d’alertes de métrique en temps quasi réel sur les données Log Analytics courantes.
author: harelbr
ms.author: harelbr
ms.topic: conceptual
ms.date: 06/17/2020
ms.subservice: alerts
ms.openlocfilehash: f9005e95ac902a2d09b792c5f64520d0175c7707
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694973"
---
# <a name="create-metric-alerts-for-logs-in-azure-monitor"></a>Créer des alertes de métrique de journaux d’activité dans Azure Monitor

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Azure Monitor prend en charge le [type d’alerte de métrique](./alerts-metric-near-real-time.md), qui présente certains avantages par rapport aux [alertes classiques](./alerts-classic-portal.md). Des métriques sont disponibles pour une [longue liste de services Azure](./metrics-supported.md). Cet article explique comment utiliser un sous-ensemble pour une ressource - `Microsoft.OperationalInsights/workspaces`.

Vous pouvez utiliser des alertes de métrique sur des journaux d’activité Log Analytics courants, extraits en tant que mesures dans le cadre des Mesures à partir de journaux d’activité, notamment des ressources dans Azure ou en local. Les solutions Log Analytics prises en charge sont répertoriées ci-dessous :

- [Les compteurs de performance](./data-sources-performance-counters.md) pour les machines Windows et Linux
- [Enregistrements de pulsations pour Agent Health](../insights/solution-agenthealth.md)
- Enregistrements de la [gestion des mises à jour](../../automation/update-management/overview.md)
- Journaux d’activité sur les [données d’événement](./data-sources-windows-events.md)

L’utilisation **d’alertes de métrique de journaux d’activité** présente de nombreux avantages par rapport à celle [d’alertes de journal](./alerts-log.md) basées sur une requête. Certains de ces avantages sont indiqués ci-dessous :

- Les alertes de métrique proposent une fonctionnalité de supervision en tant quasi réel et les alertes de métrique pour des journaux d’activité dupliquent les données de la source du journal d’activité pour garantir la même chose.
- Les alertes de métrique sont de type avec état, elles envoient des notifications uniquement quand l’alerte est déclenchée et quand elle est résolue. À l’inverse, les alertes de journal, qui sont sans état, se déclenchent à chaque intervalle si la condition d’alerte est remplie.
- Les alertes de métrique de journal fournissent plusieurs dimensions, ce qui permet d’appliquer des filtres sur des valeurs spécifiques telles que les ordinateurs, le type de système d’exploitation, etc. de façon plus simple, sans avoir besoin d’écrire des requêtes dans les analyses.

> [!NOTE]
> La métrique et/ou la dimension spécifique ne s’affichera que si des données correspondantes existent pour la période choisie. Ces métriques sont disponibles pour les clients qui ont des espaces de travail Azure Log Analytics.

## <a name="metrics-and-dimensions-supported-for-logs"></a>Métriques et dimensions prises en charge pour les journaux d’activité

 Les alertes de métrique prennent en charge la génération d’alertes pour les métriques qui utilisent des dimensions. Vous pouvez utiliser les dimensions pour filtrer votre métrique au niveau approprié. La liste complète des métriques prises en charge pour les journaux d’activité des [espaces de travail Log Analytics](./metrics-supported.md#microsoftoperationalinsightsworkspaces) est répertoriée, pour toutes les solutions prises en charge.

> [!NOTE]
> Pour afficher une métrique prise en charge extraite d’un espace de travail Log Analytics via [Azure Monitor – Métriques](./metrics-charts.md), une alerte de métrique de journal doit être créée sur cette métrique spécifique. Les dimensions choisies dans l’alerte de métrique pour journaux apparaissent uniquement pour l’exploration via Azure Monitor – Métriques.

## <a name="creating-metric-alert-for-log-analytics"></a>Création d’alerte de métrique pour Log Analytics

Les données de métrique des journaux d’activité courants sont acheminées avant leur traitement dans Log Analytics, dans Azure Monitor - Métriques. Cela permet aux utilisateurs d’exploiter les fonctionnalités de la plateforme Métrique, ainsi que les alertes de métrique, et notamment de profiter de la génération d’alertes à une fréquence de 1 minute.
Vous trouverez ci-dessous les méthodes de création d’une alerte de métrique pour des journaux d’activité.

## <a name="prerequisites-for-metric-alert-for-logs"></a>Conditions préalables pour la création d’une alerte de métrique pour des journaux d’activité

Avant que la métrique pour des journaux d’activité rassemblés sur des données Log Analytics fonctionne, les éléments suivants doivent être configurés et disponibles :

1. **Espace de travail Log Analytics actif** : un espace de travail Log Analytics actif doit être présent. Pour plus d’informations, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../learn/quick-create-workspace.md).
2. **Agent configuré pour l’espace de travail Log Analytics** : l’agent doit être configuré pour les machines virtuelles Azure et/ou les machines virtuelles locales, afin d’envoyer des données à l’espace de travail Log Analytics utilisé dans l’étape précédente. Pour plus d’informations, consultez [Présentation des agents Azure pour surveiller les machines virtuelles Azure](./agents-overview.md).
3. **Solutions Log Analytics prises en charge installées** : Une solution Log Analytics doit être configurée et doit envoyer des données à l’espace de travail Log Analytics. Les solutions prises en charge sont les [compteurs de performances pour Windows et Linux](./data-sources-performance-counters.md), les [enregistrements de pulsation pour Agent Health](../insights/solution-agenthealth.md), la [gestion des mises à jour](../../automation/update-management/overview.md) et les [données d’événement](./data-sources-windows-events.md).
4. **Solutions Log Analytics configurées pour envoyer des journaux d’activité** : une solution Log Analytics doit avoir les journaux d’activité/données correspondant aux [métriques prises en charge pour les espaces de travail Log Analytics](./metrics-supported.md#microsoftoperationalinsightsworkspaces) activés. Par exemple, le compteur *% Available Memory* (% de mémoire disponible) doit d’abord être configuré dans la solution [Compteurs de performances](./data-sources-performance-counters.md).

## <a name="configuring-metric-alert-for-logs"></a>Configuration d’une alerte de métrique pour des journaux d’activité

 Vous pouvez créer et gérer des alertes de métrique à l’aide du portail Azure, de modèles Resource Manager, de l’API REST, de PowerShell et d’Azure CLI. Étant donné que les alertes de métrique pour les journaux d’activité sont une variante des alertes de métrique, une fois que les conditions préalables sont remplies, vous pouvez créer une alerte de métrique pour les journaux d’activité de l’espace de travail Log Analytics spécifié. Toutes les caractéristiques et fonctionnalités des [alertes de métrique](./alerts-metric-near-real-time.md) sont également applicables aux alertes de métrique pour les journaux d’activité, notamment le schéma de charge utile, les limites de quota applicables et les prix facturés.

Pour obtenir des exemples et des informations détaillées, consultez [Création et gestion des alertes de métrique](./alerts-metric.md). Pour les alertes de métrique pour des journaux d’activité, suivez les instructions relatives à la gestion des alertes de métrique et vérifiez les points suivants :

- La cible de l’alerte de métrique est un *espace de travail Log Analytics* valide.
- Le signal choisi pour l’alerte de métrique pour l’*espace de travail Log Analytics* sélectionné est de type **Métrique**.
- Filtrage d’une ressource ou de conditions spécifiques à l’aide de filtres de dimension ; les métriques de journaux d’activité sont multidimensionnelles.
- Lors de la configuration de *Signal Logic* (Logique du signal), une alerte unique peut être créée pour s’étendre à plusieurs valeurs de dimension (comme Ordinateur).
- Si vous n’utilisez **pas** le portail Azure pour créer une alerte de métrique pour l’*espace de travail Log Analytics* sélectionné, l’utilisateur doit commencer par créer manuellement une règle explicite pour convertir les données de journal d’activité en métrique à l’aide d’[Azure Monitor - Règles de requête planifiées](/rest/api/monitor/scheduledqueryrules).

> [!NOTE]
> Lors de la création d’une alerte de métrique pour l’espace de travail Log Analytics via le portail Azure, la règle correspondante pour la conversion de données de journal d’activité en métrique par le biais d’[Azure Monitor - Règles de requête planifiées](/rest/api/monitor/scheduledqueryrules) est automatiquement créée en arrière-plan, *sans qu’une action ou intervention de l’utilisateur ne soit nécessaire*. Pour créer une alerte de métrique pour des journaux d’activité autrement qu’avec le Portail Azure, consultez la section [Modèle de ressource pour les alertes de métrique pour des journaux d’activité](#resource-template-for-metric-alerts-for-logs) sur les exemples de méthodes de création d’une règle de conversion de journal d’activité en métrique basée sur ScheduledQueryRule avant la création d’une alerte de métrique, sinon il n’y aura pas de données pour l’alerte de métrique sur les journaux d’activité créée.

## <a name="resource-template-for-metric-alerts-for-logs"></a>Modèle de ressource pour les alertes de métrique pour des journaux d’activité

Comme indiqué précédemment, le processus de création des alertes de métrique à partir des journaux d’activité est double :

1. Création d’une règle pour l’extraction de métriques de journaux d’activité pris en charge à l’aide de l’API scheduledQueryRule
2. Créer une alerte de métrique pour une métrique extraite d’un journal d’activité (à l’étape 1) et de l’espace de travail Log Analytics en tant que ressource cible

### <a name="metric-alerts-for-logs-with-static-threshold"></a>Alertes de métrique pour les journaux d’activité avec un seuil statique

Pour obtenir le même résultat, vous pouvez utiliser l’exemple de modèle Azure Resource Manager ci-dessous, quand la création d’une alerte de métrique de seuil statique dépend de la réussite de la création de la règle d’extraction de métriques à partir de journaux d’activité par le biais de scheduledQueryRule.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log conversion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example: /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterThan",
            "allowedValues": [
                "Equals",
                "NotEquals",
                "GreaterThan",
                "GreaterThanOrEqual",
                "LessThan",
                "LessThanOrEqual"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "threshold": {
            "type": "string",
            "defaultValue": "0",
            "metadata": {
                "description": "The threshold value at which the alert is activated."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],
                            "operator": "[parameters('operator')]",
                            "threshold" : "[parameters('threshold')]",
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"
                    }
                ]
            }
        }
    ]
}
```

Supposons que le code JSON ci-dessus soit enregistré sous le nom metricfromLogsAlertStatic.json, nous pouvons l’associer à un fichier JSON de paramètre pour la création basée sur le modèle de ressource. Vous trouverez ci-dessous un exemple de fichier JSON de paramètre :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule" 
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterThan"
        },
        "threshold":{
            "value": "1"
        },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }
}
```

En supposant que le fichier de paramètre ci-dessus est enregistré sous le nom metricfromLogsAlertStatic.parameters.json, vous pouvez créer une alerte de métrique pour des journaux d’activité à l’aide du [modèle de ressource pour la création dans le portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

Vous pouvez également utiliser la commande Azure PowerShell ci-dessous :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlertStatic.json TemplateParameterFile metricfromLogsAlertStatic.parameters.json
```

Ou déployez le modèle de ressource avec Azure CLI :

```azurecli
az deployment group create --resource-group myRG --template-file metricfromLogsAlertStatic.json --parameters @metricfromLogsAlertStatic.parameters.json
```

### <a name="metric-alerts-for-logs-with-dynamic-thresholds"></a>Alertes de métrique pour les journaux d’activité avec des seuils dynamiques

Pour obtenir le même résultat, vous pouvez utiliser l’exemple de modèle Azure Resource Manager ci-dessous, quand la création d’une alerte de métrique de seuil dynamique dépend de la réussite de la création de la règle d’extraction de métriques à partir de journaux d’activité par le biais de scheduledQueryRule.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log conversion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example: /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterOrLessThan",
            "allowedValues": [
                "GreaterThan",
                "LessThan",
                "GreaterOrLessThan"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "alertSensitivity": {
            "type": "string",
            "defaultValue": "Medium",
            "allowedValues": [
                "High",
                "Medium",
                "Low"
            ],
            "metadata": {
                "description": "Tunes how 'noisy' the Dynamic Thresholds alerts will be: 'High' will result in more alerts while 'Low' will result in fewer alerts."
            }
        },
        "numberOfEvaluationPeriods": {
            "type": "string",
            "defaultValue": "4",
            "metadata": {
                "description": "The number of periods to check in the alert evaluation."
            }
        },
        "minFailingPeriodsToAlert": {
            "type": "string",
            "defaultValue": "3",
            "metadata": {
                "description": "The number of unhealthy periods to alert on (must be lower or equal to numberOfEvaluationPeriods)."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "criterionType": "DynamicThresholdCriterion",
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],
                            "operator": "[parameters('operator')]",
                            "alertSensitivity": "[parameters('alertSensitivity')]",
                            "failingPeriods": {
                                "numberOfEvaluationPeriods": "[parameters('numberOfEvaluationPeriods')]",
                                "minFailingPeriodsToAlert": "[parameters('minFailingPeriodsToAlert')]"
                            },
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"
                    }
                ]
            }
        }
    ]
}
```

Supposons que le code JSON ci-dessus soit enregistré sous le nom metricfromLogsAlertDynamic.json, nous pouvons l’associer à un fichier JSON de paramètre pour la création basée sur le modèle de ressource. Vous trouverez ci-dessous un exemple de fichier JSON de paramètre :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule"
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterOrLessThan"
          },
          "alertSensitivity": {
              "value": "Medium"
          },
          "numberOfEvaluationPeriods": {
              "value": "4"
          },
          "minFailingPeriodsToAlert": {
              "value": "3"
          },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }
}
```

En supposant que le fichier de paramètre ci-dessus est enregistré sous le nom metricfromLogsAlertDynamic.parameters.json, vous pouvez créer une alerte de métrique pour les journaux d’activité à l’aide du [modèle de ressource pour la création dans le portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

Vous pouvez également utiliser la commande Azure PowerShell ci-dessous :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlertDynamic.json TemplateParameterFile metricfromLogsAlertDynamic.parameters.json
```

Ou déployez le modèle de ressource avec Azure CLI :

```azurecli
az deployment group create --resource-group myRG --template-file metricfromLogsAlertDynamic.json --parameters @metricfromLogsAlertDynamic.parameters.json
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [alertes de métrique](alerts-metric.md).
- En savoir plus sur les [alertes de journal dans Azure](./alerts-unified-log.md).
- En savoir plus sur les [alertes dans Azure](alerts-overview.md).