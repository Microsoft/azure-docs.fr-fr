---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/16/2020
ms.author: larryfr
ms.openlocfilehash: 95422ace63c9f79583de5a2c86d777ad4d0a8834
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027089"
---
Les entrées dans le document `deploymentconfig.json` correspondent aux paramètres pour [AksWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?view=azure-ml-py). Le tableau suivant décrit le mappage entre les entités dans le document JSON et les paramètres de la méthode :

| Entité JSON | Paramètre de méthode | Description |
| ----- | ----- | ----- |
| `computeType` | N/D | La cible de calcul. Pour AKS, la valeur doit être `aks`. |
| `autoScaler` | N/D | Contient les éléments de configuration pour la mise à l’échelle automatique. Consultez le tableau de mise à l’échelle automatique. |
| &emsp;&emsp;`autoscaleEnabled` | `autoscale_enabled` | Indique s’il faut activer la mise à l’échelle automatique pour le service Web. Si `numReplicas` = `0`, `True` ; sinon, `False`. |
| &emsp;&emsp;`minReplicas` | `autoscale_min_replicas` | Nombre minimal de conteneurs à utiliser lors de la mise à l’échelle automatique de ce service web. Par défaut, `1`. |
| &emsp;&emsp;`maxReplicas` | `autoscale_max_replicas` | Nombre maximal de conteneurs à utiliser lors de la mise à l’échelle automatique de ce service web. Par défaut, `10`. |
| &emsp;&emsp;`refreshPeriodInSeconds` | `autoscale_refresh_seconds` | Fréquence à laquelle la mise à l’échelle automatique tente de mettre à l’échelle ce service web. Par défaut, `1`. |
| &emsp;&emsp;`targetUtilization` | `autoscale_target_utilization` | Utilisation cible (en pourcentage sur 100) que la mise à l’échelle automatique doit tenter de gérer pour ce service web. Par défaut, `70`. |
| `dataCollection` | N/D | Contient les éléments de configuration pour la collecte de données. |
| &emsp;&emsp;`storageEnabled` | `collect_model_data` | Indique s’il faut activer la collecte des données de modèle pour le service Web. Par défaut, `False`. |
| `authEnabled` | `auth_enabled` | Indique s’il faut ou non activer l’authentification de clé pour le service web. `tokenAuthEnabled` et `authEnabled` ne peuvent pas tous deux avoir la valeur `True`. Par défaut, `True`. |
| `tokenAuthEnabled` | `token_auth_enabled` | Indique s’il faut ou non activer l’authentification de jeton pour le service web. `tokenAuthEnabled` et `authEnabled` ne peuvent pas tous deux avoir la valeur `True`. Par défaut, `False`. |
| `containerResourceRequirements` | N/D | Conteneur pour les entités UC et mémoire. |
| &emsp;&emsp;`cpu` | `cpu_cores` | Nombre de cœurs de processeur à allouer pour ce service web. Par défaut, `0.1` |
| &emsp;&emsp;`memoryInGB` | `memory_gb` | Quantité de mémoire (en Go) à allouer à ce service web. Par défaut, `0.5` |
| `appInsightsEnabled` | `enable_app_insights` | Indique s’il faut activer la journalisation Application Insights pour le service Web. Par défaut, `False`. |
| `scoringTimeoutMs` | `scoring_timeout_ms` | Délai d’expiration à appliquer pour les appels de scoring au service web. Par défaut, `60000`. |
| `maxConcurrentRequestsPerContainer` | `replica_max_concurrent_requests` | Nombre maximal de demandes simultanées par nœud pour ce service web. Par défaut, `1`. |
| `maxQueueWaitMs` | `max_request_wait_time` | Durée maximale pendant laquelle une demande est conservée dans la file d’attente (en millisecondes) avant qu’une erreur 503 soit renvoyée. Par défaut, `500`. |
| `numReplicas` | `num_replicas` | Nombre de conteneurs à allouer pour ce service web. Pas de valeur par défaut. Si ce paramètre n’est pas défini, la mise à l’échelle automatique est activée par défaut. |
| `keys` | N/D | Contient les éléments de configuration pour les clés. |
| &emsp;&emsp;`primaryKey` | `primary_key` | Clé d’authentification principale à utiliser pour ce service web |
| &emsp;&emsp;`secondaryKey` | `secondary_key` | Clé d’authentification secondaire à utiliser pour ce service web |
| `gpuCores` | `gpu_cores` | Nombre de cœurs GPU (par réplica de conteneur) à allouer pour ce service web. 1 constitue la valeur par défaut. Prend en charge uniquement les valeurs numériques entières. |
| `livenessProbeRequirements` | N/D | Contient les éléments de configuration pour les exigences de probe liveness. |
| &emsp;&emsp;`periodSeconds` | `period_seconds` | Fréquence (en secondes) d’exécution de probe liveness. La valeur par défaut est 10 secondes. La valeur minimale est 1. |
| &emsp;&emsp;`initialDelaySeconds` | `initial_delay_seconds` | Nombre de secondes après le démarrage du conteneur avant le lancement des probes liveness. La valeur par défaut est 310 |
| &emsp;&emsp;`timeoutSeconds` | `timeout_seconds` | Nombre de secondes après lequel la probe liveness expire. La valeur par défaut est de 2 secondes. La valeur minimale est 1 |
| &emsp;&emsp;`successThreshold` | `success_threshold` | Nombre minimal de réussites consécutives pour que la probe liveness soit considérée comme réussie après avoir échoué. La valeur par défaut est de 1. La valeur minimale est 1. |
| &emsp;&emsp;`failureThreshold` | `failure_threshold` | Quand un Pod démarre et que la probe liveness échoue, Kubernetes essaie FailureThreshold times avant d’abandonner. La valeur par défaut est 3. La valeur minimale est 1. |
| `namespace` | `namespace` | Espace de noms Kubernetes dans lequel le service web est déployé. Jusqu’à 63 caractères alphanumériques ('a'-'z', '0'-'9') et le trait d’union ('-'). Le premier et le dernier caractères ne peuvent pas être des traits d’union. |

Le code JSON suivant est un exemple de configuration de déploiement à utiliser avec l’interface CLI :

```json
{
    "computeType": "aks",
    "autoScaler":
    {
        "autoscaleEnabled": true,
        "minReplicas": 1,
        "maxReplicas": 3,
        "refreshPeriodInSeconds": 1,
        "targetUtilization": 70
    },
    "dataCollection":
    {
        "storageEnabled": true
    },
    "authEnabled": true,
    "containerResourceRequirements":
    {
        "cpu": 0.5,
        "memoryInGB": 1.0
    }
}
```