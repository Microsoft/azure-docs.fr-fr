---
title: Configurer la supervision de GPU avec Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article explique comment configurer la supervision de clusters Kubernetes constitués de nœuds équipés de GPU NVIDIA et AMD à l’aide d’Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 03/27/2020
ms.openlocfilehash: e391117ab57211aa5d178d11c27b934b4ccd37f8
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905585"
---
# <a name="configure-gpu-monitoring-with-azure-monitor-for-containers"></a>Configurer la supervision de GPU avec Azure Monitor pour conteneurs

Depuis sa version *ciprod03022019*, l’agent intégré d’Azure Monitor pour conteneurs prend désormais en charge la supervision de l’utilisation de GPU (cartes graphiques) sur les nœuds de cluster Kubernetes compatibles GPU et supervise les pods/conteneurs demandant et utilisant des ressources GPU.

## <a name="supported-gpu-vendors"></a>Fabricants de GPU compatibles

Azure Monitor pour conteneurs prend en charge la supervision des clusters de GPU des fabricants de GPU suivants :

- [NVIDIA](https://developer.nvidia.com/kubernetes-gpu)

- [AMD](https://github.com/RadeonOpenCompute/k8s-device-plugin)

Azure Monitor pour conteneurs lance automatiquement la supervision de l’utilisation de GPU sur les nœuds ainsi que celle des demandes GPU en pods et en charges de travail, en collectant les métriques suivantes toutes les 60 secondes et en les stockant dans la table **InsightMetrics**.

>[!NOTE]
>Après le provisionnement d’un cluster avec les nœuds GPU, assurez-vous que le [pilote GPU](../../aks/gpu-cluster.md) est installé, comme exigé par AKS pour exécuter des charges de travail GPU. Azure Monitor pour conteneurs collecte les métriques GPU via des pods de pilote GPU s’exécutant dans le nœud. 

|Nom de métrique |Dimension de la métrique (balises) |Description |
|------------|------------------------|------------|
|containerGpuDutyCycle |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, gpuModel, gpuVendor|Pourcentage de temps durant lequel le GPU est resté occupé/en cours de traitement pour un conteneur au cours de la période d’échantillonnage passée (60 secondes). Le cycle d’utilisation est un nombre compris entre 1 et 100. |
|containerGpuLimits |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName |Chaque conteneur peut spécifier des limites pour un ou plusieurs GPU. Il n’est pas possible de demander ou de limiter une partie d’un GPU. |
|containerGpuRequests |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName |Chaque conteneur peut demander un ou plusieurs GPU. Il n’est pas possible de demander ou de limiter une partie d’un GPU.|
|containerGpumemoryTotalBytes |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, gpuModel, gpuVendor |Quantité de mémoire de GPU en octets à disposition pour un conteneur spécifique. |
|containerGpumemoryUsedBytes |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, gpuModel, gpuVendor |Quantité de mémoire de GPU en octets utilisée par un conteneur spécifique. |
|nodeGpuAllocatable |container.azm.ms/clusterId, container.azm.ms/clusterName, gpuVendor |Nombre de GPU dans un nœud pouvant être utilisés par Kubernetes. |
|nodeGpuCapacity |container.azm.ms/clusterId, container.azm.ms/clusterName, gpuVendor |Nombre total de GPU présents dans un nœud. |

## <a name="gpu-performance-charts"></a>Graphiques de performances de GPU 

Azure Monitor pour conteneurs inclut des graphiques préconfigurés pour les métriques listées dans le tableau précédent sous la forme d’un classeur de GPU pour chaque cluster. Si vous souhaitez obtenir une description des classeurs Azure Monitor pour conteneurs, consultez [Classeurs Azure Monitor pour conteneurs](container-insights-reports.md).

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Utiliser des GPU pour les charges de travail nécessitant beaucoup de ressources système sur Azure Kubernetes Service](../../aks/gpu-cluster.md) (AKS) pour savoir comment déployer un cluster AKS constitué de nœuds avec GPU.

- Apprenez-en davantage sur les [Tailles de machine virtuelle à GPU optimisé dans Microsoft Azure](../../virtual-machines/sizes-gpu.md).

- Consultez [Prise en charge des GPU dans Kubernetes](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/) pour en savoir plus sur la prise en charge expérimentale de Kubernetes de la gestion de GPU sur un ou plusieurs nœuds d’un cluster.
