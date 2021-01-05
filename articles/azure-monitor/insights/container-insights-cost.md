---
title: Coût de l’analyse via Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article décrit le coût d’analyse des métriques et données d’inventaire collectées par Azure Monitor pour conteneurs, afin d’aider les clients à gérer leur utilisation et les frais associés.
ms.topic: conceptual
ms.date: 05/29/2020
ms.openlocfilehash: 81a20f564af68c3da6d63394e4cffe7caed91b46
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903215"
---
# <a name="understand-monitoring-costs-for-azure-monitor-for-containers"></a>Compréhension des coûts d’analyse via Azure Monitor pour conteneurs

Cet article fournit des conseils de tarification associés à Azure Monitor pour conteneurs, afin de vous aider à comprendre les éléments suivants :

* Comment estimer les coûts au préalable avant d’activer cette fonction

* Comment mesurer les coûts après l’activation de Microsoft Azure Monitor pour conteneurs et ce, pour un ou plusieurs conteneurs

* Comment contrôler la collection des données et réduire les coûts

Azure Monitor Logs collecte, indexe et stocke les données générées par votre cluster Kubernetes. 

Le modèle de tarification Azure Monitor repose principalement sur la quantité de données ingérées par jour dans votre espace de travail Log Analytics (en gigaoctets). De ce fait, le coût d’un espace de travail Log Analytics n’est pas seulement fonction du volume des données collectées, mais du fait qu’il dépend aussi du plan sélectionné et de la durée de stockage des données générées par vos clusters.

>[!NOTE]
>Toutes les tailles et tous les prix sont uniquement fournis à des fins d’estimation. Reportez-vous à la page relative à la [tarification](https://azure.microsoft.com/pricing/details/monitor/) Azure Monitor pour obtenir les prix les plus récents, en fonction de votre modèle de tarification Azure Monitor Log Analytics et de la région Azure.

Voici un résumé des types de données collectées à partir d’un cluster Kubernetes avec Azure Monitor pour conteneurs et de leur influence sur les coûts. Ces données peuvent être personnalisées selon vos besoins :

- Journaux stdout et stderr de chaque conteneur analysé dans chaque espace de noms Kubernetes du cluster

- Variables d’environnement de chaque conteneur analysé dans le cluster

- Tâches/pods Kubernetes terminés dans le cluster qui ne nécessitent pas d’analyse

- Activer le scraping des métriques Prometheus

- [Collection des journaux de diagnostic](../../aks/view-master-logs.md) des nœuds Kubernetes principaux de votre cluster AKS, pour l’analyse des données des journaux générés par les principaux composants (comme *kube-apiserver* et *kube-controller-manager*).

## <a name="what-is-collected-from-kubernetes-clusters"></a>Éléments collectés à partir des clusters Kubernetes

Azure Monitor pour conteneurs comprend un ensemble prédéfini de métriques et d’éléments d’inventaire collectés, qui sont écrits sous forme de données de journaux dans votre espace de travail Log Analytics. Toutes les métriques répertoriées ci-dessous sont collectées toutes les minutes par défaut.

### <a name="node-metrics-collected"></a>Métriques relatives aux nœuds collectées

La liste suivante répertorie les 24 métriques par nœud qui sont collectées :

- cpuUsageNanoCores
- cpuCapacityNanoCores
- cpuAllocatableNanoCores
- memoryRssBytes
- memoryWorkingSetBytes
- memoryCapacityBytes
- memoryAllocatableBytes
- restartTimeEpoch
- used (disk)
- free (disk)
- used_percent (disk)
- io_time (diskio)
- writes (diskio)
- reads (diskio)
- write_bytes (diskio)
- write_time (diskio)
- iops_in_progress (diskio)
- read_bytes (diskio)
- read_time (diskio)
- err_in (net)
- err_out (net)
- bytes_recv (net)
- bytes_sent (net)
- Kubelet_docker_operations (kubelet)

### <a name="container-metrics"></a>Métriques des conteneurs

La liste suivante répertorie les huit métriques collectées pour chaque conteneur :

- cpuUsageNanoCores
- cpuRequestNanoCores
- cpuLimitNanoCores
- memoryRssBytes
- memoryWorkingSetBytes
- memoryRequestBytes
- memoryLimitBytes
- restartTimeEpoch

### <a name="cluster-inventory"></a>Inventaire du cluster

La liste suivante répertorie les données d’inventaire du cluster collectées par défaut :

- KubePodInventory : 1 par minute et par conteneur
- KubeNodeInventory : 1 par nœud et par minute
- KubeServices : 1 par service et par minute
- ContainerInventory : 1 par conteneur et par minute

## <a name="estimating-costs-to-monitor-your-aks-cluster"></a>Estimation des coûts d’analyse de votre cluster AKS

L’estimation ci-dessous est basée sur un cluster AKS (Azure Kubernetes Service), avec l’exemple de dimensionnement suivant. En outre, l’estimation s’applique uniquement aux métriques et données d’inventaire collectées. Pour les journaux de conteneur (stdout, stderr et les variables d’environnement), cette estimation varie en fonction des tailles des journaux générés par la charge de travail ; elles sont exclues de notre estimation.

Si vous avez activé l’analyse d’un cluster AKS configuré comme suit :

- Trois nœuds
- Deux disques par nœud
- Une interface réseau par nœud
- 20 pods (un conteneur dans chaque pod, soit 20 conteneurs au total)
- Deux espaces de noms Kubernetes
- Cinq services Kubernetes (y compris les services, les pods et l’espace de nom kube-system)
- Fréquence de collecte = 60 secondes (par défaut)

Vous pouvez voir les tables et le volume de données générées par heure dans l’espace de travail Log Analytics affecté. Pour plus d’informations sur chacune de ces tables, voir [Enregistrements de conteneur](container-insights-log-search.md#container-records).

|Table de charge de travail | Estimation de la taille (Mo/heure) |
|------|---------------|
|Perf | 12.9 |
|InsightsMetrics | 11.3 |
|KubePodInventory | 1.5 |
|KubeNodeInventory | 0,75 |
|KubeServices | 0,13 |
|ContainerInventory | 3.6 |
|KubeHealth | 0.1 |
|KubeMonAgentEvents |0.005 |

Total = 31 Mo/heure = 23,1 Go/mois (un mois = 31 jours)

À l’aide de la [tarification](https://azure.microsoft.com/pricing/details/monitor/) par défaut associée à Log Analytics, laquelle repose sur un modèle de paiement à l’utilisation, vous pouvez estimer le coût de la solution Azure Monitor par mois. Une fois la réservation de capacité ajoutée, le prix par mois sera plus élevé, en fonction de la réservation sélectionnée.

## <a name="controlling-ingestion-to-reduce-cost"></a>Contrôle de l’ingestion pour réduire les coûts

Envisageons un scénario dans lequel les différentes divisions de l’organisation partagent l’infrastructure Kubernetes et un espace de travail Log Analytics. Chaque division est séparée des autres par un espace de noms Kubernetes. Vous pouvez visualiser la quantité de données qui sont ingérées dans chaque espace de travail à l’aide du runbook **Utilisation des données**, qui est accessible à partir de la liste déroulante **Voir les classeurs** .

[![Liste déroulante Voir les classeurs](media/container-insights-cost/workbooks-dropdown.png)](media/container-insights-cost/workbooks-dropdown.png#lightbox)


Ce classeur permet de visualiser la source de vos données sans avoir à créer votre propre bibliothèque de requêtes à partir des informations de notre documentation. Ce classeur inclut des graphiques avec lesquels vous pouvez afficher des données facturables depuis les perspectives suivantes :

- Total des données facturables ingérées par chaque solution, en Go
- Données facturables ingérées par chaque journal de conteneur (journal des applications)
- Données des journaux de conteneurs facturables ingérées par chaque espace de noms Kubernetes
- Données des journaux de conteneurs facturables ingérées, par nom de cluster
- Données des journaux de conteneurs facturables ingérées, par entrée logsource
- Données de diagnostic facturables ingérées, par journal de diagnostic de nœud maître

[![Classeur Utilisation des données](media/container-insights-cost/data-usage-workbook.png)](media/container-insights-cost/data-usage-workbook.png#lightbox)

Pour en savoir plus sur la gestion des droits et des autorisations d’accès aux classeurs, voir [Contrôle d’accès](../platform/workbooks-access-control.md).

Une fois que vous avez déterminé la ou les sources qui génèrent le plus de données, ou une quantité de données supérieure à vos besoins, vous pouvez reconfigurer la collection de données. Pour plus d’informations sur la configuration de la collection des variables d’environnement et de stdout et stderr, voir [Configurer la collecte de données de l’agent pour Azure Monitor pour conteneurs](container-insights-agent-config.md).

Voici des exemples de changements que vous pouvez appliquer à votre cluster en modifiant le fichier ConfigMap pour aider à contrôler les coûts.

1. Désactivez les journaux stdout sur tous les espaces de noms du cluster en modifiant les éléments suivants dans le fichier ConfigMap :

    ```
    [log_collection_settings]       
       [log_collection_settings.stdout]          
          enabled = false
    ```

2. Désactivez la collecte des journaux stderr à partir de votre espace de noms de développement (par exemple, **dev-test**), et continuez à collecter les journaux stderr à partir d’autres espaces de noms (par exemple, **prod** et **default**) en changeant les éléments suivants dans le fichier ConfigMap :

    >[!NOTE]
    >La collection de journaux de kube-system est désactivée par défaut. Le paramètre par défaut est conservé ; l’ajout de l’espace de noms **dev-test** à la liste des espaces de noms d’exclusion est appliqué à la collection de journaux stderr.

    ```
    [log_collection_settings.stderr]          
       enabled = true          
          exclude_namespaces = ["kube-system", "dev-test"]
    ```

3. Désactivez la collection de variables d’environnement sur l’ensemble du cluster en modifiant les éléments suivants dans le fichier ConfigMap. Cela s’applique à tous les conteneurs de chaque espace de noms Kubernetes. 

    ```
    [log_collection_settings.env_var]
        enabled = false
    ```

4. Pour nettoyer les travaux terminés, spécifiez la stratégie de nettoyage dans la définition du travail en modifiant les éléments suivants dans le fichier ConfigMap :

    ```
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: pi-with-ttl
    spec:
      ttlSecondsAfterFinished: 100
    ```

Après avoir appliqué une ou plusieurs de ces modifications à votre fichier ConfigMap, consultez la section [Application du ConfigMap mis à jour](container-insights-prometheus-integration.md#applying-updated-configmap) pour savoir comment l’appliquer à votre cluster.

### <a name="prometheus-metrics-scraping"></a>Capture de métriques Prometheus

Si vous utilisez [le scraping de métriques Prometheus](container-insights-prometheus-integration.md), veillez à prendre en compte les points suivants pour limiter le nombre de métriques collectées à partir de votre cluster :

- Assurez-vous que la fréquence de scraping est définie de façon optimale (la valeur par défaut est de 60 secondes). Vous pouvez faire monter la fréquence à 15 secondes, mais vous devez vous assurer que les métriques obtenues via ce scraping sont publiées à cette fréquence. Dans le cas contraire, de nombreuses métriques en double seront scrapées et envoyées à votre espace de travail Log Analytics, selon des intervalles qui augmenteront les coûts d’ingestion de données et de rétention, mais qui présenteront moins de valeur. 

- Azure Monitor pour conteneurs prend en charge les listes d’exclusions et d’inclusions par nom de métrique. Ainsi, si vous scrapez des métriques **kubedns** dans votre cluster, il se peut que des centaines d’entre elles soient capturées par défaut ; toutefois, ce sera probablement un sous-ensemble de données qui vous intéressera. Confirmez que vous avez spécifié une liste de métriques à scraper, ou excluez-en d’autres et conservez-en quelques-unes pour les enregistrer sur le volume d’ingestion de données. Vous pouvez aisément activer le scraping et éviter d’utiliser un grand nombre de ces métriques, qui ne feront qu’ajouter des frais supplémentaires à votre facture Log Analytics.

- Lors du scraping via des annotations de pods, appliquez un filtrage par espace de noms de manière à exclure le scraping des métriques de pods des espaces de noms que vous n’utilisez pas (par exemple, **l’espace de noms dev-test**).

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment interpréter les coûts qui sont susceptibles d’être basés sur les modèles d’utilisation récents des données collectées avec Azure Monitor pour conteneurs, voir [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](../platform/manage-cost-storage.md).