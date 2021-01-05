---
title: Analyser la référence de données Azure Machine Learning | Microsoft Docs
description: Documentation de référence pour la supervision d’Azure Machine Learning. Découvrez les données et les ressources collectées et disponibles dans Azure Monitor.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 10/02/2020
ms.openlocfilehash: d231de8e262194cd7dda2462a3f54d7d531eb63c
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542080"
---
# <a name="monitoring-azure-machine-learning-data-reference"></a>Informations de référence sur l’analyse des données Azure Machine Learning

Découvrez les données et les ressources collectées par Azure Monitor à partir de votre espace de travail Azure Machine Learning. Pour plus d’informations sur la collecte et l’analyse des données de monitoring, voir [Monitoring d’Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="metrics"></a>Mesures

Cette section répertorie toutes les métriques de plateforme collectées automatiquement pour Azure Machine Learning. Le fournisseur de ressources pour ces métriques est [Microsoft.MachineLearningServices/workspaces](../azure-monitor/platform/metrics-supported.md#microsoftmachinelearningservicesworkspaces).

**Modèle**

| Métrique | Unité | Description |
| ----- | ----- | ----- |
| Déploiements de modèles ayant échoué | Count | Nombre de déploiements de modèles ayant échoué. |
| Déploiements de modèles commencés | Count | Nombre de déploiements de modèles ayant commencé. |
| Déploiements de modèles réussis | Count | Nombre de déploiements de modèles ayant réussi. |
| Enregistrements de modèles ayant échoué | Count | Nombre d’enregistrements de modèles ayant échoué. |
| Enregistrements de modèles réussis | Count | Nombre d’enregistrements de modèles ayant réussi. |

**Quota**

Les informations de quota concernent uniquement le calcul Azure Machine Learning.

| Métrique | Unité | Description |
| ----- | ----- | ----- |
| Cœurs actifs | Count | Nombre de cœurs de calcul actifs. |
| Nœuds actifs | Count | Nombre de nœuds actifs. |
| Cœurs inactifs | Count | Nombre de cœurs de calcul inactifs. |
| Nœuds inactifs | Count | Nombre de nœuds de calcul inactifs. |
| Cœurs sortants | Count | Nombre de cœurs sortants. |
| Nœuds sortants | Count | Nombre de nœuds sortants. |
| Cœurs réquisitionnés | Count | Nombre de cœurs réquisitionnés. |
| Nœuds reportés | Count | Nombre de nœuds réquisitionnés. |
| Pourcentage d’utilisation du quota | Pourcentage | Pourcentage du quota utilisé. |
| Nombre total de cœurs | Count | Nombre total de cœurs. |
| Nombre total de nœuds | Count | Nombre total de nœuds. |
| Cœurs inutilisables | Count | Nombre de cœurs inutilisables. |
| Nœuds inutilisables | Count | Nombre de nœuds inutilisables. |

**Ressource**

| Métrique | Unité | Description |
| ----- | ----- | ----- |
| CpuUtilization | Pourcentage | Pourcentage d’utilisation du processeur pour un nœud donné au cours d’une exécution/tâche. Cette métrique est publiée uniquement lorsqu’une tâche est en cours d’exécution sur un nœud. Une tâche peut utiliser un ou plusieurs nœuds. Cette métrique est publiée par nœud. |
| GpuUtilization | Pourcentage | Pourcentage d’utilisation du processeur graphique (GPU) pour un nœud donné au cours d’une exécution/tâche. Un nœud peut avoir un ou plusieurs processeurs graphiques. Cette métrique est publiée par GPU, par nœud. |

**Exécuter**

Informations sur les exécutions d’apprentissage.

| Métrique | Unité | Description |
| ----- | ----- | ----- |
| Exécutions terminées | Count | Nombre d’exécutions effectuées. |
| Exécutions ayant échoué | Count | Nombre d’exécutions ayant échoué. |
| Exécutions commencées | Count | Nombre d’exécutions commencées. |

## <a name="metric-dimensions"></a>Dimensions de métrique

Pour plus d’informations sur les dimensions de métrique, consultez [Métriques multidimensionnelles](../azure-monitor/platform/data-platform-metrics.md#multi-dimensional-metrics).

Azure Machine Learning a les dimensions suivantes associées à ses métriques.

| Dimension | Description |
| ---- | ---- |
| Nom du cluster | Nom de la ressource de cluster de calcul. Disponible pour toutes les métriques de quota. |
| Nom de la famille de machines virtuelles | Nom de la famille de machines virtuelles utilisée par le cluster. Pourcentage Disponible pour l’utilisation du quota. |
| Priorité de la machine virtuelle | Priorité de la machine virtuelle. Pourcentage Disponible pour l’utilisation du quota.
| CreatedTime | Disponible uniquement pour CpuUtilization et GpuUtilization. |
| deviceId | ID du dispositif (GPU). Disponible uniquement pour GpuUtilization. |
| NodeId | ID du nœud créé où la tâche s’exécute. Disponible uniquement pour CpuUtilization et GpuUtilization. |
| RunId | ID de l’exécution/tâche. Disponible uniquement pour CpuUtilization et GpuUtilization. |
| ComputeType | Type de calcul utilisé par l’exécution. Disponible uniquement pour les exécutions terminées, les échecs d’exécutions et les exécutions démarrées. |
| PipelineStepType | Type de [PipelineStep](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinestep?preserve-view=true&view=azure-ml-py) utilisé dans l’exécution. Disponible uniquement pour les exécutions terminées, les échecs d’exécutions et les exécutions démarrées. |
| PublishedPipelineId | ID du pipeline publié utilisé dans l’exécution. Disponible uniquement pour les exécutions terminées, les échecs d’exécutions et les exécutions démarrées. |
| RunType | Type d’exécution. Disponible uniquement pour les exécutions terminées, les échecs d’exécutions et les exécutions démarrées. |

Voici les valeurs valides de la dimension RunType :

| Valeur | Description |
| ----- | ----- |
| Expérience | Exécutions hors pipeline. |
| PipelineRun | Exécution de pipeline, parente d’une StepRun. |
| StepRun | Exécution d’une étape de pipeline. |
| ReusedStepRun | Exécution d’une étape de pipeline qui réutilise une exécution précédente. |

## <a name="activity-log"></a>Journal d’activité

Le tableau suivant répertorie les opérations relatives à Azure Machine Learning qui peuvent être créées dans le journal d’activité.

| Opération | Description |
|:---|:---|
| Crée ou met à jour un espace de travail Machine Learning | Un espace de travail a été créé ou mis à jour |
| CheckComputeNameAvailability | Vérifie si un nom de calcul est déjà utilisé |
| Crée ou met à jour les ressources de calcul | Une ressource de calcul a été créée ou mise à jour |
| Supprime les ressources de calcul | Une ressource de calcul a été supprimée |
| Afficher la liste des secrets | Sur l’opération, répertorie des secrets pour un espace de travail Machine Learning |

## <a name="resource-logs"></a>Journaux d’activité de ressources

Cette section répertorie les types de journaux de ressources que vous pouvez collecter pour l’espace de travail Azure Machine Learning.

Fournisseur et type de ressources : [Microsoft.MachineLearningServices/workspace](../azure-monitor/platform/resource-logs-categories.md#microsoftmachinelearningservicesworkspaces).

| Category | Nom d’affichage |
| ----- | ----- |
| AmlComputeClusterEvent | AmlComputeClusterEvent |
| AmlComputeClusterNodeEvent | AmlComputeClusterNodeEvent |
| AmlComputeCpuGpuUtilization | AmlComputeCpuGpuUtilization |
| AmlComputeJobEvent | AmlComputeJobEvent |
| AmlRunStatusChangedEvent | AmlRunStatusChangedEvent |

## <a name="schemas"></a>Schémas

Les schémas suivants sont utilisés par Azure Machine Learning.

### <a name="amlcomputejobevents-table"></a>Tableau AmlComputeJobEvents

| Propriété | Description |
|:--- |:---|
| TimeGenerated | Heure de génération de l’entrée du journal |
| NomOpération | Nom de l’opération associée à l’événement de journal |
| Category | Nom de l’événement de journal, AmlComputeClusterNodeEvent |
| JobId | ID du travail soumis |
| ExperimentId | ID de l’expérience |
| ExperimentName | Nom de l’expérience |
| CustomerSubscriptionId | SubscriptionId où l’expérience et le travail sont envoyés |
| WorkspaceName | Nom de l’espace de travail Machine Learning |
| ClusterName | Nom du cluster |
| ProvisioningState | État de la soumission du travail |
| ResourceGroupName | Nom du groupe de ressources |
| JobName | Nom du travail |
| ClusterId | ID du cluster |
| Type d’événement | Type de l’événement de travail. Par exemple, JobSubmitted, JobRunning, JobFailed, JobSucceeded |
| ExecutionState | État du travail (exécution). Par exemple, En file d’attente, En cours d’exécution, Réussi, Échec |
| ErrorDetails | Détails de l’erreur du travail |
| CreationApiVersion | Version de l’API utilisée pour créer le travail |
| ClusterResourceGroupName | Nom du groupe de ressources du cluster |
| TFWorkerCount | Nombre de Workers TF |
| TFParameterServerCount | Nombre de serveurs de paramètres TF |
| ToolType | Type d’outil utilisé |
| RunInContainer | Indicateur décrivant si le travail doit être exécuté à l’intérieur d’un conteneur |
| JobErrorMessage | Message détaillé de l’erreur du travail |
| NodeId | ID du nœud créé sur lequel le travail s’exécute |

### <a name="amlcomputeclusterevents-table"></a>Table AmlComputeClusterEvents

| Propriété | Description |
|:--- |:--- |
| TimeGenerated | Heure de génération de l’entrée du journal |
| NomOpération | Nom de l’opération associée à l’événement de journal |
| Category | Nom de l’événement de journal, AmlComputeClusterNodeEvent |
| ProvisioningState | État d’approvisionnement du cluster |
| ClusterName | Nom du cluster |
| ClusterType | Type du cluster |
| CreatedBy | Utilisateur qui a créé le cluster |
| CoreCount | Nombre de cœurs du cluster |
| VmSize | Taille de machine virtuelle du cluster |
| VmPriority | Priorité des nœuds créés à l’intérieur d’un cluster (Dedicated/LowPriority) |
| ScalingType | Type de mise à l’échelle du cluster (manual/auto) |
| InitialNodeCount | Nombre initial de nœuds du cluster |
| MinimumNodeCount | Nombre minimal de nœuds du cluster |
| MaximumNodeCount | Nombre maximal de nœuds du cluster |
| NodeDeallocationOption | Manière dont le nœud doit être libéré |
| Serveur de publication | Éditeur du type de cluster |
| Offre | Offre avec laquelle le cluster est créé |
| Sku | Référence SKU du nœud/de la machine virtuelle créé dans le cluster |
| Version | Version de l’image utilisée lors de la création du nœud/de la machine virtuelle |
| SubnetId | SubnetId du cluster |
| AllocationState | État d’allocation du cluster |
| CurrentNodeCount | Nombre actuel de nœuds du cluster |
| TargetNodeCount | Nombre cible de nœuds du cluster lors de la mise à l’échelle |
| Type d’événement | Type d’événement lors de la création du cluster |
| NodeIdleTimeSecondsBeforeScaleDown | Durée d’inactivité en secondes avant de descendre en puissance le cluster |
| PreemptedNodeCount | Nombre de nœuds réquisitionnés du cluster |
| IsResizeGrow | Indicateur signifiant que le cluster est en cours de montée en puissance |
| VmFamilyName | Nom de la famille de machines virtuelles des nœuds qui peuvent être créés dans le cluster |
| LeavingNodeCount | Nombre de nœuds sortants du cluster |
| UnusableNodeCount | Nombre de nœuds inutilisables du cluster |
| IdleNodeCount | Nombre de nœuds inactifs du cluster |
| RunningNodeCount | Nombre de nœuds en cours d’exécution du cluster |
| PreparingNodeCount | Nombre de nœuds en préparation du cluster |
| QuotaAllocated | Quota alloué au cluster |
| QuotaUtilized | Quota utilisé du cluster |
| AllocationStateTransitionTime | Durée de transition d’un état à un autre |
| ClusterErrorCodes | Code d’erreur reçu lors de la création ou de la mise à l’échelle du cluster |
| CreationApiVersion | Version de l’API utilisée lors de la création du cluster |

### <a name="amlcomputeclusternodeevents-table"></a>Tableau AmlComputeClusterNodeEvents

| Propriété | Description |
|:--- |:--- |
| TimeGenerated | Heure de génération de l’entrée du journal |
| NomOpération | Nom de l’opération associée à l’événement de journal |
| Category | Nom de l’événement de journal, AmlComputeClusterNodeEvent |
| ClusterName | Nom du cluster |
| NodeId | ID du nœud de cluster créé |
| VmSize | Taille de machine virtuelle du nœud |
| VmFamilyName | Famille de machines virtuelles à laquelle appartient le nœud |
| VmPriority | Priorité du nœud créé (Dedicated/LowPriority) |
| Serveur de publication | Éditeur de l’image de machine virtuelle. Par exemple, microsoft-dsvm |
| Offre | Offre associée à la création de la machine virtuelle |
| Sku | Référence SKU du nœud/de la machine virtuelle créé |
| Version | Version de l’image utilisée lors de la création du nœud/de la machine virtuelle |
| ClusterCreationTime | Heure de création du cluster |
| ResizeStartTime | Heure de début de la mise à l’échelle du cluster |
| ResizeEndTime | Heure de fin de la mise à l’échelle du cluster |
| NodeAllocationTime | Heure à laquelle le nœud a été alloué |
| NodeBootTime | Heure à laquelle le nœud a été démarré |
| StartTaskStartTime | Heure à laquelle la tâche a été affectée à un nœud et a démarré |
| StartTaskEndTime | Heure à laquelle la tâche affectée à un nœud s’est terminée |
| TotalE2ETimeInSeconds | Durée totale d’activité du nœud |


## <a name="see-also"></a>Voir aussi

- Pour une description du monitoring d’Azure Machine Learning , voir [Monitoring d’Azure Machine Learning](monitor-azure-machine-learning.md).
- Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md).
