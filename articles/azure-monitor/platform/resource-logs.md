---
title: Journaux de ressources Azure
description: Découvrez comment diffuser en continu les journaux de diagnostic Azure vers un espace de travail Log Analytics dans Azure Monitor.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 07/17/2019
ms.author: bwren
ms.subservice: logs
ms.openlocfilehash: a2f46440a4214e298bc6d2f3b9c2b5680437ead7
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522700"
---
# <a name="azure-resource-logs"></a>Journaux de ressources Azure
Les journaux de ressources Azure sont des [journaux de plateforme](platform-logs-overview.md) qui fournissent des insights sur les opérations qui ont été effectuées au sein d’une ressource Azure. Le contenu des journaux de ressources varie en fonction du service Azure et du type de ressource. Les journaux d’activité de ressources ne sont pas collectées par défaut. Vous devez créer un paramètre de diagnostic pour chaque ressource Azure afin d’envoyer ses journaux de ressources vers un espace de travail Log Analytics pour les utiliser avec les [journaux Azure Monitor](data-platform-logs.md), vers Azure Event Hubs pour les transférer à l’extérieur d’Azure ou vers un stockage Azure à des fins d’archivage.

Consultez [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](diagnostic-settings.md) pour plus d’informations sur la création d’un paramètre de diagnostic et [Déployer Azure Monitor à la bonne échelle à l’aide d’Azure Policy](../deploy-scale.md) pour plus d’informations sur l’utilisation d’Azure Policy pour créer automatiquement un paramètre de diagnostic pour chaque ressource Azure que vous créez.

## <a name="send-to-log-analytics-workspace"></a>Envoyer à l’espace de travail Log Analytics
 Envoyez des journaux de ressources à un espace de travail Log Analytics pour activer les fonctionnalités des [journaux Azure Monitor](data-platform-logs.md), qui permettent notamment de :

- Mettre en corrélation les données des journaux de ressources avec d’autres données de supervision collectées par Azure Monitor.
- Consolider les entrées de journal de plusieurs ressources, abonnements et locataires Azure en un seul endroit pour les analyser ensemble.
- Utiliser les requêtes de journal pour effectuer des analyses complexes et obtenir des insights profonds sur les données de journal.
- Utiliser des alertes de journal avec une logique d’alerte complexe.

[Créez un paramètre de diagnostic](diagnostic-settings.md) pour envoyer des journaux de ressources à un espace de travail Log Analytics. Ces données sont stockées dans des tables comme décrit dans [Structure des journaux Azure Monitor](./data-platform-logs.md). Les tables utilisées par les journaux de ressources dépendent du type de collection que la ressource utilise :

- Diagnostics Azure - Toutes les données sont écrites dans la table _AzureDiagnostics_.
- Spécifique à la ressource - Les données sont écrites dans une table individuelle pour chaque catégorie de la ressource.

### <a name="azure-diagnostics-mode"></a>Mode Diagnostics Azure 
Dans ce mode, toutes les données, quel que soit le paramètre de diagnostic, sont collectées dans la table _AzureDiagnostics_. Il s’agit de la méthode héritée qu'utilise aujourd’hui la plupart des services Azure. De nombreuses ressources envoyant des données à la même table, son schéma constitue le sur-ensemble des schémas des différents types de données collectés.

Prenons l’exemple suivant dans lequel les paramètres de diagnostic sont collectés dans le même espace de travail pour les types de données suivants :

- Journaux d'audit du service 1 (schéma constitué des colonnes A, B et C)  
- Journaux des erreurs du service 1 (schéma constitué des colonnes D, E et F)  
- Journaux d'audit du service 2 (schéma constitué des colonnes G, H et I)  

La table AzureDiagnostics se présente comme suit :  

| ResourceProvider    | Category     | Un  | B  | C  | D  | E  | F  | G  | H  | I  |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| Microsoft.Service1 | AuditLogs    | x1 | y1 | z1 |    |    |    |    |    |    |
| Microsoft.Service1 | ErrorLogs    |    |    |    | q1 | w1 | e1 |    |    |    |
| Microsoft.Service2 | AuditLogs    |    |    |    |    |    |    | j1 | k1 | l1 |
| Microsoft.Service1 | ErrorLogs    |    |    |    | q2 | w2 | e2 |    |    |    |
| Microsoft.Service2 | AuditLogs    |    |    |    |    |    |    | j3 | k3 | l3 |
| Microsoft.Service1 | AuditLogs    | x5 | y5 | z5 |    |    |    |    |    |    |
| ... |

### <a name="resource-specific"></a>Spécifique à la ressource
Dans ce mode, les tables individuelles de l’espace de travail sélectionné sont créées pour chaque catégorie sélectionnée dans le paramètre de diagnostic. Cette méthode est recommandée, car elle facilite considérablement l’utilisation des données dans les requêtes de journal, offre une meilleure détectabilité des schémas et de leur structure, améliore les performances en termes de latence d’ingestion et de temps de requête, de même que la possibilité d’accorder des droits de contrôle d'accès en fonction du rôle (RBAC) Azure sur une table spécifique. À termes, tous les services Azure migreront vers le mode Spécifique à la ressource. 

L’exemple ci-dessus se traduirait par la création de trois tables :
 
- Table *Service1AuditLogs* comme suit :

    | Fournisseur de ressources | Category | Un | B | C |
    | -- | -- | -- | -- | -- |
    | Service1 | AuditLogs | x1 | y1 | z1 |
    | Service1 | AuditLogs | x5 | y5 | z5 |
    | ... |

- Table *Service1ErrorLogs* comme suit :  

    | Fournisseur de ressources | Category | D | E | F |
    | -- | -- | -- | -- | -- | 
    | Service1 | ErrorLogs |  q1 | w1 | e1 |
    | Service1 | ErrorLogs |  q2 | w2 | e2 |
    | ... |

- Table *Service2AuditLogs* comme suit :  

    | Fournisseur de ressources | Category | G | H | I |
    | -- | -- | -- | -- | -- |
    | Service2 | AuditLogs | j1 | k1 | l1|
    | Service2 | AuditLogs | j3 | k3 | l3|
    | ... |



### <a name="select-the-collection-mode"></a>Sélectionner le mode de collecte
La plupart des ressources Azure écrivent des données dans l’espace de travail dans **Diagnostics Azure** ou le **mode Spécifique à la ressource**, sans vous laisser le choix. Pour plus d’informations sur le mode utilisé, consultez la [documentation propre à chaque service](./resource-logs-schema.md). À terme, tous les services Azure utiliseront le mode Spécifique à la ressource. Dans le cadre de cette transition, certaines ressources vous permettront de sélectionner un mode dans le paramètre de diagnostic. Spécifiez le mode Spécifique à la ressource pour les nouveaux paramètres de diagnostic, car cela facilite la gestion des données et vous évitera peut-être des migrations complexes par la suite.
  
   ![Sélecteur du mode Paramètres de diagnostic](media/resource-logs-collect-workspace/diagnostic-settings-mode-selector.png)

> [!NOTE]
> Consultez [Exemples de modèle Resource Manager pour les paramètres de diagnostic dans Azure Monitor](../samples/resource-manager-diagnostic-settings.md#diagnostic-setting-for-recovery-services-vault) pour un exemple de paramétrage de diagnostic avec un modèle Resource Manager.


Vous pouvez modifier un paramètre de diagnostic existant sur le mode Spécifique à la ressource. Dans ce cas, les données déjà collectées sont conservées dans la table _AzureDiagnostics_ jusqu’à leur suppression conformément au paramètre de conservation pour l'espace de travail. Les nouvelles données sont collectées dans la table dédiée. Utilisez l’opérateur [union](/azure/kusto/query/unionoperator) pour interroger les données dans les deux tables.

Consultez régulièrement le blog [Mises à jour Azure](https://azure.microsoft.com/updates/) pour vous tenir informé des annonces relatives aux services Azure prenant en charge le mode Spécifique à la ressource.

### <a name="column-limit-in-azurediagnostics"></a>Limite de colonnes dans AzureDiagnostics
Dans les journaux d'activité Azure Monitor, les propriétés sont limitées à 500. Une fois cette limite atteinte, toutes les lignes contenant des données avec plus de 500 propriétés sont supprimées au moment de l'ingestion. La table *AzureDiagnostics* est particulièrement sensible à cette limite car elle comprend les propriétés de tous les services Azure qui y écrivent.

Si vous collectez des journaux de ressources à partir de plusieurs services, _AzureDiagnostics_ peut dépasser cette limite et certaines données n’y seront pas consignées. En attendant que tous les services Azure prennent en charge le mode Spécifique à la ressources, il vous est conseillé de configurer les ressources de manière à ce qu'elles écrivent dans plusieurs espaces de travail afin d'éviter d’atteindre la limite de 500 colonnes.

### <a name="azure-data-factory"></a>Azure Data Factory
Du fait de son ensemble détaillé de journaux, Azure Data Factory est un service connu pour écrire un grand nombre de colonnes et peut amener _AzureDiagnostics_ à dépasser sa limite. Pour tous les paramètres de diagnostic configurés avant l’activation du mode Spécifique à la ressource, une colonne est créée pour chaque paramètre utilisateur portant un nom unique par rapport à une activité. D’autres colonnes sont créées en raison de la nature détaillée des entrées et des sorties d’activité.
 
Vous devez migrer vos journaux pour utiliser le mode Spécifique à la ressource dans les meilleurs délais. Si vous ne pouvez pas le faire immédiatement, une alternative temporaire consiste à isoler les journaux Azure Data Factory dans leur propre espace de travail afin de minimiser les risques d'impact de ceux-ci sur d'autres types de journaux collectés dans vos espaces de travail.


## <a name="send-to-azure-event-hubs"></a>Envoyer à Azure Event Hubs
Envoyez les journaux de ressources à un hub d’événements pour les envoyer à l’extérieur d’Azure, par exemple à une solution SIEM ou à d’autres solutions analytiques de journaux. Les journaux de ressources en provenance de hubs d’événements sont consommés au format JSON avec un élément `records` contenant les enregistrements de chaque charge utile. Le schéma dépend du type de ressource, comme décrit dans [Schéma commun et spécifique au service pour les journaux de ressources Azure](resource-logs-schema.md).

Voici des exemples de données de sortie provenant d’Event Hubs pour un journal de ressource :

```json
{
    "records": [
        {
            "time": "2019-07-15T18:00:22.6235064Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330013509921957/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Error",
            "operationName": "Microsoft.Logic/workflows/workflowActionCompleted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T17:58:55.048482Z",
                "endTime": "2016-07-15T18:00:22.4109204Z",
                "status": "Failed",
                "code": "BadGateway",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330013509921957",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "29a9862f-969b-4c70-90c4-dfbdc814e413",
                    "clientTrackingId": "08587330013509921958"
                }
            }
        },
        {
            "time": "2019-07-15T18:01:15.7532989Z",
            "workflowId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/JOHNKEMTEST/PROVIDERS/MICROSOFT.LOGIC/WORKFLOWS/JOHNKEMTESTLA/RUNS/08587330012106702630/ACTIONS/SEND_EMAIL",
            "category": "WorkflowRuntime",
            "level": "Information",
            "operationName": "Microsoft.Logic/workflows/workflowActionStarted",
            "properties": {
                "$schema": "2016-04-01-preview",
                "startTime": "2016-07-15T18:01:15.5828115Z",
                "status": "Running",
                "resource": {
                    "subscriptionId": "00000000-0000-0000-0000-000000000000",
                    "resourceGroupName": "JohnKemTest",
                    "workflowId": "243aac67fe904cf195d4a28297803785",
                    "workflowName": "JohnKemTestLA",
                    "runId": "08587330012106702630",
                    "location": "westus",
                    "actionName": "Send_email"
                },
                "correlation": {
                    "actionTrackingId": "042fb72c-7bd4-439e-89eb-3cf4409d429e",
                    "clientTrackingId": "08587330012106702632"
                }
            }
        }
    ]
}
```

## <a name="send-to-azure-storage"></a>Envoyer à Stockage Azure
Envoyez les journaux de ressources à un stockage Azure pour les y conserver à des fins d’archivage. Une fois que vous avez créé le paramètre de diagnostic, un conteneur de stockage est créé dans le compte de stockage dès qu’un événement se produit dans les catégories de journaux activées. Les objets blob présents dans le conteneur utilisent la convention d’affectation de noms suivante :

```
insights-logs-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/RESOURCEGROUPS/{resource group name}/PROVIDERS/{resource provider name}/{resource type}/{resource name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Par exemple, l’objet blob d’un groupe de sécurité réseau peut être nommé de la façon suivante :

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json
```

Chaque objet blob PT1H.json contient un objet blob d’événements JSON qui se sont produits pendant l’heure spécifiée dans l’URL de l’objet blob (par exemple, h=12). Pendant l’heure en cours, les événements sont ajoutés au fichier PT1H.json à mesure qu’ils se produisent. La valeur de minute (m=00) est toujours 00, car les événements du journal de ressource sont divisés en objets blob par heure.

Dans le fichier PT1H.json, chaque événement est stocké au format suivant. Il utilise un schéma de niveau supérieur courant mais est unique pour chaque service Azure comme décrit dans [Schéma des journaux de ressource](./resource-logs-schema.md).

``` JSON
{"time": "2016-07-01T00:00:37.2040000Z","systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1","category": "NetworkSecurityGroupRuleCounter","resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG","operationName": "NetworkSecurityGroupCounters","properties": {"vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}","subnetPrefix": "10.3.0.0/24","macAddress": "000123456789","ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp","direction": "In","type": "allow","matchedConnections": 1988}}
```

> [!NOTE]
> Les journaux de plateforme sont écrits dans le stockage d’objets blob à l’aide de [lignes JSON](http://jsonlines.org/), où chaque événement correspond à une ligne et le caractère de nouvelle ligne indique un nouvel événement. Ce format a été implémenté en novembre 2018. Avant cette date, les journaux étaient écrits dans le stockage d’objets blob sous forme de tableau d’enregistrements json, comme décrit dans l’article [Préparer le changement de format des journaux de plateforme Azure Monitor archivés dans un compte de stockage](resource-logs-blob-format.md).


## <a name="next-steps"></a>Étapes suivantes

* [Découvrez plus d’informations sur les journaux de ressource](platform-logs-overview.md)
* [Créez des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](diagnostic-settings.md).