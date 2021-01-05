---
title: Configuration d’Azure Monitor pour la collecte de données de l’agent de conteneurs | Microsoft Docs
description: Cet article décrit comment configurer Azure Monitor pour que l’agent de conteneurs contrôle stdout/stderr et la collecte des journaux de variables d’environnement.
ms.topic: conceptual
ms.date: 10/09/2020
ms.openlocfilehash: f21b841bc129012b684d2a1c59eb72989fe9e0e0
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890494"
---
# <a name="configure-agent-data-collection-for-azure-monitor-for-containers"></a>Configurer la collecte de données de l’agent pour Azure Monitor pour conteneurs

Azure Monitor pour conteneurs collecte stdout, stderr et des variables d’environnement des charges de travail de conteneur qui sont déployées sur des clusters Kubernetes managés à partir de l’agent conteneurisé. Vous pouvez configurer les paramètres de collecte de données de l’agent en créant une ConfigMaps Kubernetes personnalisée pour contrôler cette expérience. 

Cet article montre comment créer la ConfigMap et configurer la collecte de données selon vos besoins.

>[!NOTE]
>Pour Azure Red Hat OpenShift, un fichier de modèle ConfigMap est créé dans l’espace de noms *openshift-azure-logging* . 
>

## <a name="configmap-file-settings-overview"></a>Vue d’ensemble des paramètres de fichier ConfigMap

Un fichier ConfigMap de modèle est fourni. Vous pouvez le modifier facilement avec vos personnalisations sans avoir à le créer à partir de zéro. Avant de commencer, vous devez consulter la documentation Kubernetes sur [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) et vous familiariser avec la manière de créer, de configurer et de déployer ConfigMaps. Cela vous permet de filtrer stderr et stdout par espace de noms ou dans l’ensemble du cluster et les variables d’environnement pour les conteneurs en cours d’exécution sur tous les nœuds/pods du cluster.

>[!IMPORTANT]
>La version minimale de l’agent prise en charge pour collecter des variables stdout, stderr et environnementales à partir de charges de travail de conteneur est ciprod06142019 ou ultérieure. Pour vérifier la version de votre agent, sous l’onglet **Nœud** , sélectionnez un nœud, puis dans la valeur note du volet Propriétés de la propriété **Balise d’image de l’agent** . Pour plus d’informations sur les versions de l’agent et le contenu de chaque version, consultez les [notes de publication de l’agent](https://github.com/microsoft/Docker-Provider/tree/ci_feature_prod).

### <a name="data-collection-settings"></a>Paramètres de collecte de données

Le tableau suivant décrit les paramètres que vous pouvez configurer pour contrôler la collecte de données :

| Clé | Type de données | Valeur | Description |
|--|--|--|--|
| `schema-version` | Chaîne (respecte la casse) | v1 | Il s’agit de la version de schéma utilisée par l’agent<br> lors de l’analyse de ce ConfigMap.<br> Actuellement, la version prise en charge est v1.<br> La modification de cette valeur n’est pas prise en charge et est<br> rejetée lors de l’évaluation de ConfigMap. |
| `config-version` | String |  | Prend en charge la possibilité d’effectuer le suivi de la version de ce fichier de configuration dans votre système/dépôt de contrôle de code source.<br> Le nombre maximal de caractères autorisé est 10 et tous les autres caractères sont tronqués. |
| `[log_collection_settings.stdout] enabled =` | Boolean | True ou False | Ce paramètre contrôle si la collecte de journaux de conteneur stdout est activée. Quand la valeur est `true` et qu’aucun espace de noms n’est exclu pour la collection de journaux stdout<br> (paramètre `log_collection_settings.stdout.exclude_namespaces` ci-dessous), les journaux stdout sont collectés à partir de tous les conteneurs sur tous les pods/nœuds du cluster. Si elle n’est pas spécifiée dans ConfigMaps,<br> la valeur par défaut est `enabled = true`. |
| `[log_collection_settings.stdout] exclude_namespaces =` | String | Tableau séparé par des virgules | Tableau d’espaces de noms Kubernetes pour lesquels aucun journal stdout n’est collecté. Ce paramètre est effectif uniquement<br> `log_collection_settings.stdout.enabled`<br> s’il a la valeur `true`.<br> Si elle n’est pas spécifiée dans ConfigMap, la valeur par défaut est<br> `exclude_namespaces = ["kube-system"]`. |
| `[log_collection_settings.stderr] enabled =` | Boolean | True ou False | Ce paramètre contrôle si la collecte de journaux de conteneur stderr est activée.<br> Quand la valeur est `true` et qu’aucun espace de noms n’est exclu pour la collection de journaux stdout<br> (paramètre `log_collection_settings.stderr.exclude_namespaces`), les journaux stderr sont collectés à partir de tous les conteneurs sur tous les pods/nœuds du cluster.<br> Si elle n’est pas spécifiée dans ConfigMaps, la valeur par défaut est<br> `enabled = true`. |
| `[log_collection_settings.stderr] exclude_namespaces =` | String | Tableau séparé par des virgules | Tableau d’espaces de noms Kubernetes pour lesquels aucun journal stderr n’est collecté.<br> Ce paramètre est effectif uniquement<br> `log_collection_settings.stdout.enabled` est défini sur `true`.<br> Si elle n’est pas spécifiée dans ConfigMap, la valeur par défaut est<br> `exclude_namespaces = ["kube-system"]`. |
| `[log_collection_settings.env_var] enabled =` | Boolean | True ou False | Ce paramètre contrôle la collecte de variables d’environnement<br> sur tous les pods/nœuds du cluster.<br> Sa valeur par défaut est `enabled = true` quand elle n’est pas spécifiée<br> dans ConfigMaps.<br> Si la collecte de variables d’environnement est globalement activée, vous pouvez la désactiver pour un conteneur spécifique<br> en attribuant à la variable d’environnement<br> `AZMON_COLLECT_ENV` la valeur **false** avec un paramètre Dockerfile ou dans le [fichier de configuration du pod](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) sous la section **env:** .<br> Si la collecte des variables d’environnement est désactivée globalement, vous ne pouvez pas activer la collecte pour un conteneur spécifique (autrement dit, la seule substitution applicable au niveau du conteneur consiste à désactiver la collecte lorsqu’elle est déjà activée globalement). |
| `[log_collection_settings.enrich_container_logs] enabled =` | Boolean | True ou False | Ce paramètre contrôle l’enrichissement du journal de conteneur pour remplir les valeurs de propriété Name et Image<br> pour chaque enregistrement de journal écrit dans la table ContainerLog de tous les journaux de conteneurs du cluster.<br> La valeur par défaut est `enabled = false` lorsqu’elle n’est pas spécifiée dans ConfigMap. |
| `[log_collection_settings.collect_all_kube_events]` | Boolean | True ou False | Ce paramètre autorise la collecte des événements Kube de tous les types.<br> Par défaut, les événements Kube de type *Normal* ne sont pas collectés. Quand ce paramètre a la valeur `true`, les événements *Normal* ne sont plus filtrés et tous les événements sont collectés.<br> Par défaut, il a la valeur `false`. |

### <a name="metric-collection-settings"></a>Paramètres de collecte des métriques

Le tableau suivant décrit les paramètres que vous pouvez configurer pour contrôler la collecte des métriques :

| Clé | Type de données | Valeur | Description |
|--|--|--|--|
| `[metric_collection_settings.collect_kube_system_pv_metrics] enabled =` | Boolean | True ou False | Ce paramètre permet de collecter les métriques d’utilisation de volume persistant dans l’espace de noms kube-system. Par défaut, les métriques d’utilisation des volumes persistants ayant des revendications de volume persistant dans l’espace de noms kube-system ne sont pas collectées. Lorsque ce paramètre est défini sur `true`, les métriques d’utilisation de volume persistant sont collectées pour tous les espaces de noms. Par défaut, il a la valeur `false`. |

ConfigMaps est une liste globale et il ne peut y avoir qu’un seul élément ConfigMap appliqué à l’agent. Vous ne pouvez pas avoir un autre élément ConfigMaps qui annule les collectes.

## <a name="configure-and-deploy-configmaps"></a>Configurer et déployer ConfigMaps

Procédez comme suit pour configurer et déployer votre fichier de configuration ConfigMap dans votre cluster.

1. Téléchargez le [modèle de fichier YAML ConfigMap](https://aka.ms/container-azm-ms-agentconfig) et enregistrez-le sous le nom container-azm-ms-agentconfig.yaml. 

   > [!NOTE]
   > Cette étape n’est pas nécessaire si vous utilisez Azure Red Hat OpenShift, car le modèle ConfigMap existe déjà sur le cluster.

2. Modifiez le fichier YAML ConfigMap avec vos personnalisations pour collecter les variables stdout, stderr et/ou d’environnement. Si vous modifiez le fichier yaml ConfigMap pour Azure Red Hat OpenShift, commencez par exécuter la commande `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` pour ouvrir le fichier dans un éditeur de texte.

    - Pour exclure des espaces de noms spécifiques pour la collecte de journaux stdout, configurez la clé/valeur à l’aide de l’exemple suivant : `[log_collection_settings.stdout] enabled = true exclude_namespaces = ["my-namespace-1", "my-namespace-2"]`.
    
    - Pour désactiver la collecte de variables d’environnement pour un conteneur spécifique, définissez la clé/valeur `[log_collection_settings.env_var] enabled = true` pour activer la collecte de variables globalement, puis suivez les étapes décrites [ici](container-insights-manage-agent.md#how-to-disable-environment-variable-collection-on-a-container) pour terminer la configuration pour le conteneur spécifique.
    
    - Pour désactiver la collecte de journaux stderr au niveau du cluster, configurez la clé/valeur à l’aide de l’exemple suivant : `[log_collection_settings.stderr] enabled = false`.

3. Pour les clusters autres qu’Azure Red Hat OpenShift, créez le fichier ConfigMap en exécutant la commande kubectl `kubectl apply -f <configmap_yaml_file.yaml>` sur des clusters autres qu’Azure Red Hat OpenShift. 
    
    Exemple : `kubectl apply -f container-azm-ms-agentconfig.yaml`. 

    Pour Azure Red Hat OpenShift, enregistrez vos modifications dans l’éditeur.

Quelques minutes peuvent être nécessaires pour que la modification de configuration soit effective. Ensuite, tous les pods omsagent du cluster redémarrent. Le redémarrage s’effectue de façon progressive pour tous les pods omsagent. Tous ne redémarrent pas en même temps. Lorsque les redémarrages sont terminés, un message similaire à celui-ci s’affiche avec les résultats : `configmap "container-azm-ms-agentconfig" created`.

## <a name="verify-configuration"></a>Vérifier la configuration

Pour vérifier que la configuration a été correctement appliquée à un cluster autre qu’Azure Red Hat OpenShift, exécutez la commande `kubectl logs omsagent-fdf58 -n kube-system` et examinez les journaux d’un pod d’agent. S’il existe des erreurs de configuration à partir de pods osmagent, la sortie affiche des erreurs similaires à ce qui suit :

``` 
***************Start Config Processing******************** 
config::unsupported/missing config schema version - 'v21' , using defaults
```

Les erreurs liées à l’application de modifications de configuration sont également disponibles pour consultation. Les options suivantes sont disponibles pour effectuer un dépannage supplémentaire des modifications de configuration :

- À partir de journaux d’activité d’un pod d’agent à l’aide de la même commande `kubectl logs`. 

    >[!NOTE]
    >Cette commande n’est pas applicable à un cluster Azure Red Hat OpenShift.
    > 

- À partir de journaux d’activité dynamiques. Les journaux dynamiques affichent des erreurs similaires à ce qui suit :

    ```
    config::error::Exception while parsing config map for log collection/env variable settings: \nparse error on value \"$\" ($end), using defaults, please check config map for errors
    ```

- À partir de la table **KubeMonAgentEvents** dans votre espace de travail Log Analytics. Les données sont envoyées toutes les heures avec la gravité de *l’erreur* pour les erreurs de configuration. S’il n’y a pas d’erreur, l’entrée de la table contient des données indiquant une gravité *Informations* , laquelle ne signale aucune erreur. La propriété **Balises** contient plus d’informations sur le pod et l’ID de conteneur où l’erreur s’est produite, ainsi que sur la première occurrence, la dernière occurrence et le nombre d’occurrences au cours de la dernière heure.

- Avec Azure Red Hat OpenShift, examinez les journaux omsagent et vérifiez dans la table **ContainerLog** si la collecte des journaux d’openshift-azure-logging est activée.

Une fois que vous avez corrigé les erreurs dans ConfigMap sur les clusters autres qu’Azure Red Hat OpenShift, enregistrez le fichier yaml et appliquez le ConfigMaps mis à jour en exécutant la commande `kubectl apply -f <configmap_yaml_file.yaml`. Pour Azure Red Hat OpenShift, modifiez et enregistrez le ConfigMaps mis à jour en exécutant la commande :

``` bash
oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

## <a name="applying-updated-configmap"></a>Application du ConfigMap mis à jour

Si vous avez déjà déployé un ConfigMap sur un cluster autre qu’Azure Red Hat OpenShift et que vous souhaitez le mettre à jour avec une configuration plus récente, vous pouvez modifier le fichier ConfigMap que vous utilisiez précédemment, puis l’appliquer à l’aide de la même commande qu’auparavant, `kubectl apply -f <configmap_yaml_file.yaml`. Pour Azure Red Hat OpenShift, modifiez et enregistrez le ConfigMaps mis à jour en exécutant la commande :

``` bash
oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

Quelques minutes peuvent être nécessaires pour que la modification de configuration soit effective. Ensuite, tous les pods omsagent du cluster redémarrent. Le redémarrage s’effectue de façon progressive pour tous les pods omsagent. Tous ne redémarrent pas en même temps. Lorsque les redémarrages sont terminés, un message similaire à celui-ci s’affiche avec les résultats : `configmap "container-azm-ms-agentconfig" updated`.

## <a name="verifying-schema-version"></a>Vérification de la version du schéma

Les versions de schéma de configuration pris en charge sont disponibles en tant qu’annotation de pod (versions de schéma) sur le pod omsagent. Vous pouvez les voir avec la commande kubectl suivante : `kubectl describe pod omsagent-fdf58 -n=kube-system`

La sortie qui s’affiche est similaire à ce qui suit avec les versions de schéma d’annotation :

```
    Name:           omsagent-fdf58
    Namespace:      kube-system
    Node:           aks-agentpool-95673144-0/10.240.0.4
    Start Time:     Mon, 10 Jun 2019 15:01:03 -0700
    Labels:         controller-revision-hash=589cc7785d
                    dsName=omsagent-ds
                    pod-template-generation=1
    Annotations:    agentVersion=1.10.0.1
                  dockerProviderVersion=5.0.0-0
                    schema-versions=v1 
```

## <a name="next-steps"></a>Étapes suivantes

- Azure Monitor pour conteneurs n’inclut pas d’un ensemble prédéfini d’alertes. Pour savoir comment créer les alertes recommandées pour une utilisation élevée du processeur et de la mémoire, consultez [Créer des alertes de performances avec Azure Monitor pour conteneurs](./container-insights-log-alerts.md).

- Une fois l’analyse activée pour collecter l’utilisation des ressources et l’intégrité de votre Azure Kubernetes Service ou de vos clusters hybrides et charges de travail s’y exécutant, découvrez [comment utiliser](container-insights-analyze.md) Azure Monitor pour les conteneurs.

- Consultez les [exemples de requêtes de journal](container-insights-log-search.md#search-logs-to-analyze-data) pour voir les requêtes prédéfinies et des exemples permettant d’évaluer ou de personnaliser la génération d’alertes, la visualisation ou l’analyse de vos clusters.