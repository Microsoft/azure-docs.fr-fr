---
title: Stratégies réseau Azure Kubernetes | Microsoft Docs
description: Découvrez plus en détail les stratégies réseau Kubernetes pour sécuriser votre cluster Kubernetes.
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/25/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: 36e5bb33b7d555c3b457b63f94d9032ff390e6cb
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342312"
---
# <a name="azure-kubernetes-network-policies-overview"></a>Vue d’ensemble des stratégies réseau Azure Kubernetes

Les stratégies réseau permettent une micro-segmentation des pods tout comme les groupes de sécurité réseau (NSG) fournissent une micro-segmentation des machines virtuelles. L’implémentation du gestionnaire de stratégies réseau Azure (également appelé Azure NPM) prend en charge la spécification standard de la stratégie réseau Kubernetes. Vous pouvez utiliser des étiquettes pour sélectionner un groupe de pods et définir une liste de règles d’entrée et de sortie pour filtrer le trafic vers et depuis ces pods. Découvrez plus en détail les stratégies réseau Kubernetes dans la [documentation Kubernetes](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

![Vue d’ensemble des stratégies réseau Kubernetes](./media/kubernetes-network-policies/kubernetes-network-policies-overview.png)

L’implémentation d’Azure NPM fonctionne conjointement avec Azure CNI qui fournit l’intégration de réseau virtuel aux conteneurs. NPM est uniquement pris en charge sur Linux. L’implémentation applique le filtrage du trafic en configurant des règles d’autorisation et de refus d’adresses IP dans Linux IPTables en fonction des stratégies définies. Ces règles sont regroupées à l’aide de Linux IPSets.

## <a name="planning-security-for-your-kubernetes-cluster"></a>Planification de la sécurité pour votre cluster Kubernetes
Quand vous implémentez la sécurité de votre cluster, utilisez des groupes de sécurité réseau (NSG) pour filtrer le trafic entrant et sortant de votre sous-réseau de cluster (trafic Nord-Sud). Utilisez Azure NPM pour le trafic entre les pods de votre cluster (trafic Est-Ouest).

## <a name="using-azure-npm"></a>Utilisation d’Azure NPM
Azure NPM peut être utilisé dans les méthodes suivantes pour fournir une micro-segmentation des pods.

### <a name="azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS)
NPM est disponible en mode natif dans AKS et peut être activé au moment de la création du cluster. Pour plus d’informations, consultez [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/use-network-policies).

### <a name="aks-engine"></a>AKS-engine
AKS-Engine est un outil qui génère un modèle Azure Resource Manager pour déployer un cluster Kubernetes dans Azure. La configuration du cluster est spécifiée dans un fichier JSON qui est passé à l’outil lors de la génération du modèle. Pour connaître la liste complète des paramètres de cluster pris en charge et leur description, consultez Moteur de Microsoft Azure Container Service - Définition du cluster.

Pour activer les stratégies sur des clusters déployés à l’aide d’acs-engine, définissez la valeur du paramètre networkPolicy dans le fichier de définition de cluster sur « azure ».

#### <a name="example-configuration"></a>Exemple de configuration

L’exemple de fichier JSON de configuration ci-dessous crée un réseau virtuel et un sous-réseau, et y déploie un cluster Kubernetes avec Azure CNI. Nous vous recommandons d’utiliser le « Bloc-notes » pour modifier le fichier JSON. 
```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
         "networkPolicy": "azure"
       }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "<specify a cluster name>",
      "vmSize": "Standard_D2s_v3"
    },
    "agentPoolProfiles": [
      {
        "name": "agentpool",
        "count": 2,
        "vmSize": "Standard_D2s_v3",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
   "linuxProfile": {
      "adminUsername": "<specify admin username>",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "<cut and paste your ssh key here>"
          }
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "<enter the client ID of your service principal here >",
      "secret": "<enter the password of your service principal here>"
    }
  }
}

```
### <a name="do-it-yourself-diy-kubernetes-clusters-in-azure"></a>Clusters Kubernetes DIY (à déployer soi-même) dans Azure
 Pour les clusters DIY, installez d’abord le plug-in CNI et activez-le sur chaque machine virtuelle d’un cluster. Pour obtenir des instructions détaillées, consultez [Déployer le plug-in pour un cluster Kubernetes que vous déployez vous-même](deploy-container-networking.md#deploy-plug-in-for-a-kubernetes-cluster).

Une fois le cluster déployé, exécutez la commande `kubectl` suivante pour télécharger et appliquer le *DaemonSet* Azure NPM au cluster.

  ```
  kubectl apply -f https://raw.githubusercontent.com/Azure/acs-engine/master/parts/k8s/addons/kubernetesmasteraddons-azure-npm-daemonset.yaml

  ```
La solution est également open source et le code est disponible dans le [dépôt Azure Container Networking](https://github.com/Azure/azure-container-networking/tree/master/npm).

## <a name="monitor-and-visualize-network-configurations-with-azure-npm"></a>Analyser et visualiser les configurations réseau avec Azure NPM
Azure NPM comprend des métriques Prometheus informatives qui vous permettent d’analyser et de mieux comprendre vos configurations. Il fournit des visualisations intégrées dans le portail Azure ou les laboratoires Grafana. Vous pouvez commencer à collecter ces métriques en utilisant Azure Monitor ou un serveur Prometheus.

### <a name="benefits-of-azure-npm-metrics"></a>Avantages des métriques Azure NPM
Auparavant, les utilisateurs pouvaient uniquement en savoir plus sur leur configuration réseau avec la commande `iptables -L` exécutée à l’intérieur d’un nœud de cluster, ce qui génère une sortie détaillée et difficile à comprendre. Les métriques NPM offrent les avantages suivants en ce qui concerne les stratégies réseau, les règles IPTables et IPSets.
- Fournissent un aperçu de la relation entre les trois éléments ci-dessus et une dimension de temps pour déboguer une configuration.
- Nombre d’entrées dans tous les IPSets et chaque IPSet.
- Temps nécessaire pour appliquer une stratégie avec une granularité de niveau IPTable/IPSet.
 
### <a name="supported-metrics"></a>Métriques prises en charge
Voici la liste des métriques prises en charge :

|Nom de métrique |Description  |Type de métrique Prometheus  |Étiquettes  |
|---------|---------|---------|---------|
|`npm_num_policies`     |nombre de stratégies réseau          |Jauge         |-         |
|`npm_num_iptables_rules`     | nombre de règles IPTables     | Jauge        |-         |         
|`npm_num_ipsets`     |nombre d’IPSets         |Jauge            |-         |
|`npm_num_ipset_entries`     |nombre d’entrées d’adresse IP dans tous les IPSets         |Jauge         |-         |
|`npm_add_policy_exec_time`     |runtime pour l’ajout d’une stratégie réseau         |Résumé         |quantile (0,5, 0,9 ou 0,99)         |
|`npm_add_iptables_rule_exec_time`     |runtime pour l’ajout d’une règle IPTables         |Résumé         |quantile (0,5, 0,9 ou 0,99)         |
|`npm_add_ipset_exec_time`     |runtime pour l’ajout d’un IPSet         |Résumé         |quantile (0,5, 0,9 ou 0,99)         |
|`npm_ipset_counts` (avancé)     |nombre d’entrées dans chaque IPSet individuel         |GaugeVec         |définir le nom et le hachage         |

Les différents niveaux de quantile dans les métriques « exec_time » vous aident à faire la différence entre le scénario général et le pire des scénarios.

Il existe également une métrique « exec_time_count » et une métrique « exec_time_sum » pour chaque métrique Summary « exec_time ».

Les métriques peuvent être scrapées via Azure Monitor pour conteneurs ou via Prometheus.

### <a name="setup-for-azure-monitor"></a>Configuration d’Azure Monitor
La première étape consiste à activer Azure Monitor pour conteneurs sur votre cluster Kubernetes. Les étapes sont accessibles dans [Vue d’ensemble d’Azure Monitor pour conteneurs](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-overview). Une fois que vous avez activé Azure Monitor pour conteneurs, configurez l’objet [ConfigMap Azure Monitor pour conteneurs](https://aka.ms/container-azm-ms-agentconfig) pour activer l’intégration NPM et la collecte des métriques NPM Prometheus. L’objet ConfigMap Azure Monitor pour conteneurs contient une section ```integrations``` avec des paramètres permettant de collecter les métriques NPM. Ces paramètres sont désactivés par défaut dans l’objet ConfigMap. L’activation du paramètre de base ```collect_basic_metrics = true``` collectera les métriques NPM de base. L’activation du paramètre avancé ```collect_advanced_metrics = true``` collectera des métriques avancées en plus des métriques de base. 

Après avoir modifié l’objet ConfigMap, enregistrez-le localement et appliquez-le à votre cluster comme suit.

```kubectl apply -f container-azm-ms-agentconfig.yaml``` Vous trouverez ci-dessous un extrait de code de l’[objet ConfigMap Azure Monitor pour conteneurs](https://aka.ms/container-azm-ms-agentconfig), qui montre l’intégration NPM activée avec la collecte des métriques avancées.
```
integrations: |-
    [integrations.azure_network_policy_manager]
        collect_basic_metrics = false
        collect_advanced_metrics = true
```
Les métriques avancées sont facultatives et leur activation activera automatiquement la collecte des métriques de base. Les métriques avancées incluent actuellement uniquement `npm_ipset_counts`.

En savoir plus sur les [paramètres de collecte Azure Monitor pour conteneurs dans ConfigMap](https://aka.ms/azmon-containers-agent-collection-settings-doc).

### <a name="visualization-options-for-azure-monitor"></a>Options de visualisation pour Azure Monitor
Une fois la collecte de métriques NPM activée, vous pouvez afficher les métriques dans le portail Azure à l’aide de Container Insights ou dans Grafana.

#### <a name="viewing-in-azure-portal-under-insights-for-the-cluster"></a>Affichage dans le portail Azure sous Insights pour le cluster
Ouvrez le portail Azure. Une fois que vous êtes dans la section Insights de votre cluster, accédez à « Classeurs » et ouvrez « Configuration du gestionnaire de stratégies réseau (NPM) ».

Outre l’affichage du classeur (images ci-dessous), vous pouvez également interroger directement les métriques Prometheus dans « Journaux » sous la section Insights. Par exemple, cette requête retourne toutes les métriques collectées :
| where TimeGenerated > ago(5h) | where Name contains "npm_".

Vous pouvez également interroger directement Log Analytics pour les métriques. En savoir plus à ce sujet avec [Prise en main des requêtes Log Analytics](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-log-search). 

#### <a name="viewing-in-grafana-dashboard"></a>Affichage dans le tableau de bord Grafana
Configurez votre serveur Grafana et configurez une source de données Log Analytics comme décrit [ici](https://grafana.com/grafana/plugins/grafana-azure-monitor-datasource). Ensuite, importez le [tableau de bord Grafana avec un serveur principal Log Analytics](https://grafana.com/grafana/dashboards/10956) dans vos laboratoires Grafana.

Le tableau de bord comporte des objets visuels similaires à ceux du classeur Azure. Vous pouvez ajouter des panneaux au graphique et visualiser les métriques NPM à partir du tableau InsightsMetrics.

### <a name="setup-for-prometheus-server"></a>Configuration du serveur Prometheus
Certains utilisateurs peuvent choisir de collecter les métriques avec un serveur Prometheus au lieu d’Azure Monitor pour conteneurs. Il vous suffit d’ajouter deux travaux à votre configuration de scrapage pour collecter les métriques NPM.

Pour installer un serveur Prometheus simple, ajoutez ce référentiel Helm sur votre cluster :
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo update
```
Ajoutez ensuite un serveur :
```
helm install prometheus stable/prometheus -n monitoring \
--set pushgateway.enabled=false,alertmanager.enabled=false, \
--set-file extraScrapeConfigs=prometheus-server-scrape-config.yaml
```
Où `prometheus-server-scrape-config.yaml` se compose de :
```
- job_name: "azure-npm-node-metrics"
  metrics_path: /node-metrics
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - source_labels: [__address__]
    action: replace
    regex: ([^:]+)(?::\d+)?
    replacement: "$1:10091"
    target_label: __address__
- job_name: "azure-npm-cluster-metrics"
  metrics_path: /cluster-metrics
  kubernetes_sd_configs:
  - role: service
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    regex: kube-system
    action: keep
  - source_labels: [__meta_kubernetes_service_name]
    regex: npm-metrics-cluster-service
    action: keep
# Comment from here to the end to collect advanced metrics: number of entries for each IPSet
  metric_relabel_configs:
  - source_labels: [__name__]
    regex: npm_ipset_counts
    action: drop
```


Vous pouvez également remplacer le travail `azure-npm-node-metrics` par le contenu ci-dessous ou l’incorporer dans une tâche préexistante pour les pods Kubernetes :
```
- job_name: "azure-npm-node-metrics-from-pod-config"
  metrics_path: /node-metrics
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    regex: kube-system
    action: keep
  - source_labels: [__meta_kubernetes_pod_annotationpresent_azure_npm_scrapeable]
    action: keep
  - source_labels: [__address__]
    action: replace
    regex: ([^:]+)(?::\d+)?
    replacement: "$1:10091"
    target_label: __address__
```

### <a name="visualization-options-for-prometheus"></a>Options de visualisation pour Prometheus
Lors de l’utilisation d’un serveur Prometheus, seul le tableau de bord Grafana est pris en charge. 

Si vous ne l’avez pas déjà fait, configurez votre serveur Grafana et configurez une source de données Prometheus. Ensuite, importez notre [tableau de bord Grafana avec un serveur principal Prometheus](https://grafana.com/grafana/dashboards/13000) dans vos laboratoires Grafana.

Les objets visuels de ce tableau de bord sont identiques à ceux du tableau de bord avec un serveur principal Container Insights/Log Analytics.

### <a name="sample-dashboards"></a>Sample Dashboards
Voici quelques exemples de tableau de bord pour les métriques NPM dans Container Insights (CI) et Grafana.

#### <a name="ci-summary-counts"></a>Nombres récapitulatifs CI
![Nombres récapitulatifs dans le classeur Azure](media/kubernetes-network-policies/workbook-summary-counts.png)

#### <a name="ci-counts-over-time"></a>Nombres dans le temps CI
[![Nombres dans le temps dans le classeur Azure](media/kubernetes-network-policies/workbook-counts-over-time.png)](media/kubernetes-network-policies/workbook-counts-over-time.png#lightbox)

#### <a name="ci-ipset-entries"></a>Entrées IPSet CI
[![Entrées IPSet dans le classeur Azure](media/kubernetes-network-policies/workbook-ipset-entries.png)](media/kubernetes-network-policies/workbook-ipset-entries.png#lightbox)

#### <a name="ci-runtime-quantiles"></a>Quantiles du runtime CI
![Quantiles du runtime dans le classeur Azure](media/kubernetes-network-policies/workbook-runtime-quantiles.png)

#### <a name="grafana-dashboard-summary-counts"></a>Nombres récapitulatifs dans le tableau de bord Grafana
![Nombres récapitulatifs dans le tableau de bord Grafana](media/kubernetes-network-policies/grafana-summary-counts.png)

#### <a name="grafana-dashboard-counts-over-time"></a>Nombres dans le temps dans le tableau de bord Grafana
[![Nombres dans le temps dans le tableau de bord Grafana](media/kubernetes-network-policies/grafana-counts-over-time.png)](media/kubernetes-network-policies/grafana-counts-over-time.png#lightbox)

#### <a name="grafana-dashboard-ipset-entries"></a>Entrées IPSet dans le tableau de bord Grafana
[![Entrées IPSet dans le tableau de bord Grafana](media/kubernetes-network-policies/grafana-ipset-entries.png)](media/kubernetes-network-policies/grafana-ipset-entries.png#lightbox)

#### <a name="grafana-dashboard-runtime-quantiles"></a>Quantiles du runtime dans le tableau de bord Grafana
[![Quantiles du runtime dans le tableau de bord Grafana](media/kubernetes-network-policies/grafana-runtime-quantiles.png)](media/kubernetes-network-policies/grafana-runtime-quantiles.png#lightbox)



## <a name="next-steps"></a>Étapes suivantes
- Découvrez en détail [Azure Kubernetes Service](../aks/intro-kubernetes.md).
-  Découvrez en détail la [mise en réseau de conteneurs](container-networking-overview.md).
- [Déployez le plug-in](deploy-container-networking.md) pour des clusters Kubernetes ou des conteneurs Docker.

    
