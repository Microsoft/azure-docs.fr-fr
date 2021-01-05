---
title: Configurer l’intégration Azure Monitor pour conteneurs à Prometheus | Microsoft Docs
description: Cet article explique comment configurer l’agent Azure Monitor pour conteneurs, afin qu’il récupère des métriques à partir de Prometheus avec votre cluster Kubernetes.
ms.topic: conceptual
ms.date: 04/22/2020
ms.openlocfilehash: f5a9b364bc3e51307bd44d8338485f482bda6e1e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90971355"
---
# <a name="configure-scraping-of-prometheus-metrics-with-azure-monitor-for-containers"></a>Configurer la capture des métriques Prometheus avec Azure Monitor pour conteneurs

[Prometheus](https://prometheus.io/) est une solution populaire de supervision des métriques open source, elle fait partie de la [Cloud Native Computing Foundation](https://www.cncf.io/). Azure Monitor pour conteneurs offre une expérience d’intégration transparente en permettant de collecter des métriques Prometheus. Généralement, pour utiliser Prometheus vous devez configurer et gérer un serveur Prometheus avec un magasin. Par l’intégration avec Azure Monitor, un serveur Prometheus n’est pas nécessaire. Il vous suffit d’exposer le point de terminaison des métriques Prometheus via vos exportateurs ou vos pods (application) pour que l’agent conteneurisé d’Azure Monitor pour conteneurs soit en mesure d’extraire les métriques pour vous. 

![Architecture de surveillance de conteneur pour Prometheus](./media/container-insights-prometheus-integration/monitoring-kubernetes-architecture.png)

>[!NOTE]
>La version minimale de l’agent prise en charge pour la récupération des métriques Prometheus est ciprod07092019 ou une version ultérieure ; la version de l’agent pour l’écriture de la configuration et des erreurs de l’agent dans la table `KubeMonAgentEvents` est ciprod10112019. Pour Azure Red Hat OpenShift et Red Hat OpenShift v4, version d’agent ciprod04162020 ou ultérieure. 
>
>Pour plus d’informations sur les versions de l’agent et le contenu de chaque version, consultez les [notes de publication de l’agent](https://github.com/microsoft/Docker-Provider/tree/ci_feature_prod). 
>Pour vérifier la version de votre agent, sous l’onglet **Nœud**, sélectionnez un nœud, puis dans la valeur note du volet Propriétés de la propriété **Balise d’image de l’agent**.

L’extraction de métriques Prometheus est prise en charge avec les clusters Kubernetes hébergés sur :

- Azure Kubernetes Service (AKS)
- Azure Stack ou localement
- Azure Red Hat OpenShift version 3.x
- Azure Red Hat OpenShift et Red Hat OpenShift version 4.x

### <a name="prometheus-scraping-settings"></a>Paramètres de capture de Prometheus

La capture active de métriques à partir de Prometheus est effectuée de l’une des deux perspectives suivantes :

* Sur l’ensemble du cluster - URL HTTP et découverte des cibles à partir des points de terminaison listés d’un service. Par exemple, les services k8s tels que kube-dns et kube-state-metrics, et les annotations pod spécifiques à une application. Les mesures collectées dans ce contexte sont définies dans la section ConfigMap *[Prometheus data_collection_settings.cluster]* .
* À l’ensemble du nœud - URL HTTP et découverte des cibles à partir des points de terminaison listés d’un service. Les mesures collectées dans ce contexte sont définies dans la section ConfigMap *[Prometheus data_collection_settings.node]* .

| Point de terminaison | Étendue |  Exemple |
|----------|-------|---------|
| Annotation de pod | À l’ensemble du cluster | annotations : <br>`prometheus.io/scrape: "true"` <br>`prometheus.io/path: "/mymetrics"` <br>`prometheus.io/port: "8000"` <br>`prometheus.io/scheme: "http"` |
| Service Kubernetes | À l’ensemble du cluster | `http://my-service-dns.my-namespace:9100/metrics` <br>`https://metrics-server.kube-system.svc.cluster.local/metrics` |
| URL/point de terminaison | Par nœud et/ou ensemble du cluster | `http://myurl:9101/metrics` |

Quand une URL est spécifiée, Azure Monitor pour conteneurs ne capture que le point de terminaison. Quand le service Kubernetes est spécifié, le nom du service est résolu avec le serveur DNS du cluster pour obtenir l’adresse IP, puis le service résolu est capturé.

|Étendue | Clé | Type de données | Valeur | Description |
|------|-----|-----------|-------|-------------|
| À l’ensemble du cluster | | | | Spécifiez l’une des trois méthodes suivantes pour capturer les points de terminaison pour les mesures. |
| | `urls` | String | Tableau séparé par des virgules | Point de terminaison HTTP (adresse IP ou chemin d’URL valide spécifié). Par exemple : `urls=[$NODE_IP/metrics]`. ($NODE_IP est un paramètre Azure Monitor spécifique pour conteneurs et peut être utilisé à la place d’une adresse IP de nœud. Doit être tout en majuscules.) |
| | `kubernetes_services` | String | Tableau séparé par des virgules | Tableau de services Kubernetes pour la capture des mesures à partir des kube-state-metrics. Par exemple : `kubernetes_services = ["https://metrics-server.kube-system.svc.cluster.local/metrics",http://my-service-dns.my-namespace:9100/metrics]`.|
| | `monitor_kubernetes_pods` | Boolean | True ou False | Lorsque la valeur `true` est définie sur dans les paramètres du cluster, Azure Monitor de l’agent des conteneurs capture les pod Kubernetes sur l’ensemble du cluster pour les annotations Prometheus suivantes :<br> `prometheus.io/scrape:`<br> `prometheus.io/scheme:`<br> `prometheus.io/path:`<br> `prometheus.io/port:` |
| | `prometheus.io/scrape` | Boolean | True ou False | Permet la capture du pod. `monitor_kubernetes_pods` doit être défini sur `true`. |
| | `prometheus.io/scheme` | String | http ou https | La valeur par défaut est la capture de HTTP. Si nécessaire, affectez la valeur `https`. | 
| | `prometheus.io/path` | String | Tableau séparé par des virgules | Chemin d’accès de la ressource HTTP à partir duquel récupérer les mesures. Si le chemin d’accès aux mesures n’est pas `/metrics`, définissez-le avec cette annotation. |
| | `prometheus.io/port` | String | 9102 | Spécifiez le port à utiliser pour la mise en rebut. Si le port n’est pas défini, sa valeur par défaut est 9102. |
| | `monitor_kubernetes_pods_namespaces` | String | Tableau séparé par des virgules | Liste verte d’espaces de noms pour la capture des métriques à partir des pods Kubernetes.<br> Par exemple : `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]` |
| À l’ensemble du nœud | `urls` | String | Tableau séparé par des virgules | Point de terminaison HTTP (adresse IP ou chemin d’URL valide spécifié). Par exemple : `urls=[$NODE_IP/metrics]`. ($NODE_IP est un paramètre Azure Monitor spécifique pour conteneurs et peut être utilisé à la place d’une adresse IP de nœud. Doit être tout en majuscules.) |
| À l’ensemble du nœud ou du cluster | `interval` | String | 60s | La valeur par défaut de l’intervalle de collection est d’une minute (60 secondes). Vous pouvez modifier la collection pour *[prometheus_data_collection_settings. node]* et/ou pour *[prometheus_data_collection_settings. cluster]* en unités de temps telles que s, m, h. |
| À l’ensemble du nœud ou du cluster | `fieldpass`<br> `fielddrop`| String | Tableau séparé par des virgules | Vous pouvez spécifier certaines mesures à collecter ou non à partir du point de terminaison en définissant la liste Autoriser (`fieldpass`) et Interdire (`fielddrop`). Vous devez d’abord définir la liste Autoriser. |

ConfigMaps est une liste globale et il ne peut y avoir qu’un seul élément ConfigMap appliqué à l’agent. Vous ne pouvez pas avoir un autre élément ConfigMaps qui annule les collectes.

## <a name="configure-and-deploy-configmaps"></a>Configurer et déployer ConfigMaps

Effectuez les étapes suivantes pour configurer votre fichier de configuration ConfigMap pour les clusters suivants :

* Azure Kubernetes Service (AKS)
* Azure Stack ou localement
* Azure Red Hat OpenShift version 4.x et Red Hat OpenShift version 4.x

1. [Téléchargez](https://aka.ms/container-azm-ms-agentconfig) le fichier yaml ConfigMap de modèle et enregistrez-le sous le nom container-azm-ms-agentconfig.yaml.

   >[!NOTE]
   >Cette étape n’est pas nécessaire lors de l’utilisation d’Azure Red Hat OpenShift, car le modèle ConfigMap existe déjà sur le cluster.

2. Modifiez le fichier yaml ConfigMap avec vos personnalisations pour capturer les métriques Prometheus.

    >[!NOTE]
    >Si vous modifiez le fichier yaml ConfigMap pour Azure Red Hat OpenShift, commencez par exécuter la commande `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` pour ouvrir le fichier dans un éditeur de texte.

    >[!NOTE]
    >L’annotation `openshift.io/reconcile-protect: "true"` suivante doit être ajoutée sous les métadonnées de *container-azm-ms-agentconfig* dans ConfigMap pour empêcher le rapprochement. 
    >```
    >metadata:
    >   annotations:
    >       openshift.io/reconcile-protect: "true"
    >```

    - Pour effectuer la collecte des services Kubernetes sur l’ensemble du cluster, configurez le fichier ConfigMap à l’aide de l’exemple suivant.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]
        ```

    - Pour configurer la mise en rebut des métriques Prometheus à partir d’une URL spécifique dans le cluster, configurez le fichier ConfigMap à l’aide de l’exemple suivant.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        urls = ["http://myurl:9101/metrics"] ## An array of urls to scrape metrics from
        ```

    - Pour configurer la mise en rebut des métriques Prometheus à partir du DaemonSet d’un agent pour chaque nœud du cluster, configurez les éléments suivants dans le fichier ConfigMap :
    
        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings 
        [prometheus_data_collection_settings.node] 
        interval = "1m"  ## Valid time units are s, m, h. 
        urls = ["http://$NODE_IP:9103/metrics"] 
        fieldpass = ["metric_to_pass1", "metric_to_pass2"] 
        fielddrop = ["metric_to_drop"] 
        ```

        >[!NOTE]
        >$NODE_IP est un paramètre Azure Monitor spécifique pour conteneurs et peut être utilisé à la place d’une adresse IP de nœud. Il doit être tout en majuscules. 

    - Pour configurer la mise au rebut des métriques Prometheus en spécifiant une annotation Pod, procédez comme suit :

       1. Dans le fichier ConfigMap, spécifiez les éléments suivants :

            ```
            prometheus-data-collection-settings: |- 
            # Custom Prometheus metrics data collection settings
            [prometheus_data_collection_settings.cluster] 
            interval = "1m"  ## Valid time units are s, m, h
            monitor_kubernetes_pods = true 
            ```

       2. Spécifiez la configuration suivante pour les annotations Pod :

           ```
           - prometheus.io/scrape:"true" #Enable scraping for this pod 
           - prometheus.io/scheme:"http" #If the metrics endpoint is secured then you will need to set this to `https`, if not default ‘http’
           - prometheus.io/path:"/mymetrics" #If the metrics path is not /metrics, define it with this annotation. 
           - prometheus.io/port:"8000" #If port is not 9102 use this annotation
           ```
    
          Si vous souhaitez limiter la supervision à des espaces de noms spécifiques de pods ayant des annotations, par exemple inclure uniquement les pods dédiés aux charges de travail de production, définissez `monitor_kubernetes_pod` sur `true` dans ConfigMap, et ajoutez le filtre d’espace de noms `monitor_kubernetes_pods_namespaces` en précisant l’espace de noms à partir duquel capturer. Par exemple : `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]`

3. Exécutez la commande kubectl suivante : `kubectl apply -f <configmap_yaml_file.yaml>`.
    
    Exemple : `kubectl apply -f container-azm-ms-agentconfig.yaml`. 

Quelques minutes peuvent être nécessaires pour que la modification de configuration soit effective. Ensuite, tous les pods omsagent du cluster redémarrent. Le redémarrage s’effectue de façon progressive pour tous les pods omsagent. Tous ne redémarrent pas en même temps. Lorsque les redémarrages sont terminés, un message similaire à celui-ci s’affiche avec les résultats : `configmap "container-azm-ms-agentconfig" created`.

## <a name="configure-and-deploy-configmaps---azure-red-hat-openshift-v3"></a>Configurer et déployer ConfigMaps - Azure Red Hat OpenShift v3

Cette section décrit la configuration requise et les étapes à suivre pour configurer correctement votre fichier de configuration ConfigMap pour un cluster Azure Red Hat OpenShift v3.x.

>[!NOTE]
>Pour Azure Red Hat OpenShift v3.x, un fichier de modèle ConfigMap est créé dans l’espace de noms *openshift-azure-logging*. Il n’est pas configuré pour extraire activement des métriques ou recueillir des données à partir de l’agent.

### <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous êtes membre du rôle Administrateur de cluster client de votre cluster Azure Red Hat OpenShift, afin de pouvoir configurer l’agent en conteneur et les paramètres d’extraction Prometheus. Pour vérifier que vous êtes membre du groupe *osa-customer-admins*, exécutez la commande suivante :

``` bash
  oc get groups
```

La sortie doit ressembler à ce qui suit :

``` bash
NAME                  USERS
osa-customer-admins   <your-user-account>@<your-tenant-name>.onmicrosoft.com
```

Si vous êtes membre d’*osa-customer-admins*, vous devriez pouvoir lister le configmap `container-azm-ms-agentconfig` à l’aide de la commande suivante :

``` bash
oc get configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

La sortie doit ressembler à ce qui suit :

``` bash
NAME                           DATA      AGE
container-azm-ms-agentconfig   4         56m
```

### <a name="enable-monitoring"></a>Activer la supervision

Effectuez les étapes suivantes pour configurer votre fichier de configuration ConfigMap pour votre cluster Azure Red Hat OpenShift v3.x.

1. Modifiez le fichier yaml ConfigMap avec vos personnalisations pour capturer les métriques Prometheus. Le modèle ConfigMap existe déjà sur le cluster Red Hat OpenShift v3. Exécutez la commande `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` pour ouvrir le fichier dans un éditeur de texte.

    >[!NOTE]
    >L’annotation `openshift.io/reconcile-protect: "true"` suivante doit être ajoutée sous les métadonnées de *container-azm-ms-agentconfig* dans ConfigMap pour empêcher le rapprochement. 
    >```
    >metadata:
    >   annotations:
    >       openshift.io/reconcile-protect: "true"
    >```

    - Pour effectuer la collecte des services Kubernetes sur l’ensemble du cluster, configurez le fichier ConfigMap à l’aide de l’exemple suivant.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]
        ```

    - Pour configurer la mise en rebut des métriques Prometheus à partir d’une URL spécifique dans le cluster, configurez le fichier ConfigMap à l’aide de l’exemple suivant.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        urls = ["http://myurl:9101/metrics"] ## An array of urls to scrape metrics from
        ```

    - Pour configurer la mise en rebut des métriques Prometheus à partir du DaemonSet d’un agent pour chaque nœud du cluster, configurez les éléments suivants dans le fichier ConfigMap :
    
        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings 
        [prometheus_data_collection_settings.node] 
        interval = "1m"  ## Valid time units are s, m, h. 
        urls = ["http://$NODE_IP:9103/metrics"] 
        fieldpass = ["metric_to_pass1", "metric_to_pass2"] 
        fielddrop = ["metric_to_drop"] 
        ```

        >[!NOTE]
        >$NODE_IP est un paramètre Azure Monitor spécifique pour conteneurs et peut être utilisé à la place d’une adresse IP de nœud. Il doit être tout en majuscules. 

    - Pour configurer la mise au rebut des métriques Prometheus en spécifiant une annotation Pod, procédez comme suit :

       1. Dans le fichier ConfigMap, spécifiez les éléments suivants :

            ```
            prometheus-data-collection-settings: |- 
            # Custom Prometheus metrics data collection settings
            [prometheus_data_collection_settings.cluster] 
            interval = "1m"  ## Valid time units are s, m, h
            monitor_kubernetes_pods = true 
            ```

       2. Spécifiez la configuration suivante pour les annotations Pod :

           ```
           - prometheus.io/scrape:"true" #Enable scraping for this pod 
           - prometheus.io/scheme:"http" #If the metrics endpoint is secured then you will need to set this to `https`, if not default ‘http’
           - prometheus.io/path:"/mymetrics" #If the metrics path is not /metrics, define it with this annotation. 
           - prometheus.io/port:"8000" #If port is not 9102 use this annotation
           ```
    
          Si vous souhaitez limiter la supervision à des espaces de noms spécifiques de pods ayant des annotations, par exemple inclure uniquement les pods dédiés aux charges de travail de production, définissez `monitor_kubernetes_pod` sur `true` dans ConfigMap, et ajoutez le filtre d’espace de noms `monitor_kubernetes_pods_namespaces` en précisant l’espace de noms à partir duquel capturer. Par exemple : `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]`

2. Enregistrez vos modifications dans l’éditeur.

Quelques minutes peuvent être nécessaires pour que la modification de configuration soit effective. Ensuite, tous les pods omsagent du cluster redémarrent. Le redémarrage s’effectue de façon progressive pour tous les pods omsagent. Tous ne redémarrent pas en même temps. Lorsque les redémarrages sont terminés, un message similaire à celui-ci s’affiche avec les résultats : `configmap "container-azm-ms-agentconfig" created`.

Vous pouvez afficher le modèle ConfigMap mis à jour en exécutant la commande `oc describe configmaps container-azm-ms-agentconfig -n openshift-azure-logging`. 

## <a name="applying-updated-configmap"></a>Application du ConfigMap mis à jour

Si vous avez déjà déployé un ConfigMap dans votre cluster et que vous souhaitez le mettre à jour avec une configuration plus récente, vous pouvez modifier le fichier ConfigMap que vous utilisiez précédemment, puis appliquer les mêmes commandes qu’auparavant.

Pour les environnements Kubernetes suivants :

- Azure Kubernetes Service (AKS)
- Azure Stack ou localement
- Azure Red Hat OpenShift et Red Hat OpenShift version 4.x

Exécutez la commande `kubectl apply -f <configmap_yaml_file.yaml`. 

Pour un cluster Azure Red Hat OpenShift v3.x, exécutez la commande `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` pour ouvrir le fichier dans votre éditeur, puis modifiez-le et fermez-le.

Quelques minutes peuvent être nécessaires pour que la modification de configuration soit effective. Ensuite, tous les pods omsagent du cluster redémarrent. Le redémarrage s’effectue de façon progressive pour tous les pods omsagent. Tous ne redémarrent pas en même temps. Lorsque les redémarrages sont terminés, un message similaire à celui-ci s’affiche avec les résultats : `configmap "container-azm-ms-agentconfig" updated`.

## <a name="verify-configuration"></a>Vérifier la configuration

Pour vérifier que la configuration a été correctement appliquée à un cluster, utilisez la commande suivante pour passer en revue les journaux à partir d’un pod d’agent : `kubectl logs omsagent-fdf58 -n=kube-system`. 

>[!NOTE]
>Cette commande n’est pas applicable à un cluster Azure Red Hat OpenShift v3.x.
> 

S’il existe des erreurs de configuration à partir de pods osmagent, la sortie affiche des erreurs similaires à ce qui suit :

``` 
***************Start Config Processing******************** 
config::unsupported/missing config schema version - 'v21' , using defaults
```

Les erreurs liées à l’application de modifications de configuration sont également disponibles pour consultation. Les options suivantes sont disponibles pour résoudre des problèmes supplémentaires liés aux modifications de configuration et à la capture des métriques Prometheus :

- À partir des journaux d’un pod d’agent à l’aide de la même commande `kubectl logs`. 
    >[!NOTE]
    >Cette commande n’est pas applicable au cluster Azure Red Hat OpenShift.
    > 

- À partir de données actives (préversion). Les journaux de données actives (préversion) affichent des erreurs similaires à ce qui suit :

    ```
    2019-07-08T18:55:00Z E! [inputs.prometheus]: Error in plugin: error making HTTP request to http://invalidurl:1010/metrics: Get http://invalidurl:1010/metrics: dial tcp: lookup invalidurl on 10.0.0.10:53: no such host
    ```

- À partir de la table **KubeMonAgentEvents** dans votre espace de travail Log Analytics. Les données sont envoyées toutes les heures avec la gravité *Avertissement* pour les erreurs de capture, et la gravité *Erreur* pour les erreurs de configuration. S’il n’y a pas d’erreur, l’entrée de la table contient des données indiquant une gravité *Informations*, laquelle ne signale aucune erreur. La propriété **Balises** contient plus d’informations sur le pod et l’ID de conteneur où l’erreur s’est produite, ainsi que sur la première occurrence, la dernière occurrence et le nombre d’occurrences au cours de la dernière heure.

- Pour Azure Red Hat OpenShift v3.x et v4.x, vérifiez les journaux omsagent en recherchant dans la table **ContainerLog** pour vérifier si la collecte des journaux d’openshift-azure-logging est activée.

Des erreurs empêchent omsagent d’analyser le fichier, ce qui provoque son redémarrage et l’utilisation de la configuration par défaut. Une fois que vous avez corrigé les erreurs dans ConfigMap sur les clusters autres qu’Azure Red Hat OpenShift v3.x, enregistrez le fichier yaml et appliquez le modèle ConfigMaps mis à jour en exécutant la commande `kubectl apply -f <configmap_yaml_file.yaml`. 

Pour Azure Red Hat OpenShift v3.x, modifiez et enregistrez le ConfigMaps mis à jour en exécutant la commande `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging`.

## <a name="query-prometheus-metrics-data"></a>Interroger les données de métriques Prometheus

Pour afficher les métriques Prometheus récupérées par Azure Monitor ainsi que toutes les erreurs de configuration/capture signalées par l’agent, consultez [Interroger les données de métriques Prometheus](container-insights-log-search.md#query-prometheus-metrics-data) et [Interroger les erreurs de configuration ou de capture](container-insights-log-search.md#query-config-or-scraping-errors).

## <a name="view-prometheus-metrics-in-grafana"></a>Consulter les métriques Prometheus dans Grafana

Azure Monitor pour conteneurs prend en charge la consultation des métriques stockées dans votre espace de travail Log Analytics, dans les tableaux de bord Grafana. Nous avons fourni un modèle que vous pouvez télécharger à partir du [référentiel des tableaux de bord](https://grafana.com/grafana/dashboards?dataSource=grafana-azure-monitor-datasource&category=docker) de Grafana pour vous aider à démarrer, et une référence permettant de vous familiariser avec l’interrogation des données supplémentaires depuis vos clusters supervisés, pour une visualisation dans des tableaux de bord Grafana personnalisés. 

## <a name="review-prometheus-data-usage"></a>Examinez l’utilisation des données Prometheus

Pour identifier le volume d’ingestion de chaque taille de mesure en Go par jour pour déterminer si elle est élevée, la requête suivante est fournie.

```
InsightsMetrics
| where Namespace contains "prometheus"
| where TimeGenerated > ago(24h)
| summarize VolumeInGB = (sum(_BilledSize) / (1024 * 1024 * 1024)) by Name
| order by VolumeInGB desc
| render barchart
```

La sortie indiquera des résultats similaires à ce qui suit :

![Capture d’écran affichant les résultats de la requête de journal du volume d’ingestion de données](./media/container-insights-prometheus-integration/log-query-example-usage-03.png)

Pour estimer la taille de chaque mesure en Go pendant un mois pour déterminer si le volume de données reçues dans l’espace de travail est élevé, la requête suivante est fournie.

```
InsightsMetrics
| where Namespace contains "prometheus"
| where TimeGenerated > ago(24h)
| summarize EstimatedGBPer30dayMonth = (sum(_BilledSize) / (1024 * 1024 * 1024)) * 30 by Name
| order by EstimatedGBPer30dayMonth desc
| render barchart
```

La sortie indiquera des résultats similaires à ce qui suit :

![Consignez les résultats de requête du volume d’ingestion de données](./media/container-insights-prometheus-integration/log-query-example-usage-02.png)

Vous trouverez plus d’informations sur la surveillance de l’utilisation des données et l’analyse des coûts dans [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](../platform/manage-cost-storage.md).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la configuration des paramètres de collecte d’agent pour stdout, stderr et les variables d’environnement depuis des charges de travail de conteneur [ici](container-insights-agent-config.md). 
