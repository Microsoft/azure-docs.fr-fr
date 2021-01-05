---
title: Afficher les journaux d’activité de contrôleur Azure Kubernetes Service (AKS)
description: Découvrez comment activer et afficher les journaux d’activité relatifs au nœud principal Kubernetes dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 10/14/2020
ms.openlocfilehash: 59e7259ae352491bddebe054f2c34bdc810ea48a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183224"
---
# <a name="enable-and-review-kubernetes-master-node-logs-in-azure-kubernetes-service-aks"></a>Activer et consulter les journaux d’activité du nœud principal Kubernetes dans Azure Kubernetes Service (AKS)

Avec Azure Kubernetes Service (AKS), les composants principaux tels que *kube-apiserver* et *kube-controller-manager* sont fournis sous la forme d’un service administré. Vous créez et gérez les nœuds qui exécutent le *kubelet* et le runtime de conteneurs, et vous déployez vos applications par le biais du serveur d’API de l’environnement Kubernetes managés. Pour faciliter la résolution des problèmes de votre application et de vos services, vous pouvez avoir besoin d’afficher les journaux d’activité générés par ces composants principaux. Cet article vous indique comment utiliser Azure Monitor pour activer et interroger les journaux d’activité provenant des composants principaux Kubernetes.

## <a name="before-you-begin"></a>Avant de commencer

Cet article requiert l’exécution d’un cluster AKS existant dans votre compte Azure. Si vous ne disposez pas encore d’un cluster AKS, créez-en un à l’aide de l’[interface de ligne de commande Azure (Azure CLI)][cli-quickstart] ou du [portail Azure][portal-quickstart]. Les journaux d’activité Azure Monitor fonctionnent avec des clusters AKS Kubernetes RBAC, Azure RBAC et non RBAC.

## <a name="enable-resource-logs"></a>Activer les journaux de ressources

Pour faciliter la collecte et l’analyse des données provenant de plusieurs sources, les journaux d’activité Azure Monitor fournissent un langage de requêtes et un moteur d’analyse vous permettant d’obtenir des insights sur votre environnement. Un espace de travail est utilisé pour assembler et analyser les données, et peut s’intégrer à d’autres services Azure tels qu’Application Insights et Security Center. Si vous souhaitez analyser les journaux d’activité à l’aide d’une autre plateforme, vous pouvez plutôt choisir d’envoyer les journaux de ressources à un compte de stockage ou à un Event Hub Azure. Pour plus d’informations, voir [Présentation des journaux d’activité Azure Monitor][log-analytics-overview].

Les journaux Azure Monitor sont activés et gérés sur le portail Azure. Pour activer la collecte des journaux relatifs aux composants principaux Kubernetes dans votre cluster AKS, ouvrez le Portail Azure dans un navigateur web et procédez comme suit :

1. Sélectionnez le groupe de ressources de votre cluster AKS, par exemple *myResourceGroup*. Ne sélectionnez pas le groupe de ressources qui contient vos ressources de cluster AKS individuelles, comme *MC_myResourceGroup_myAKSCluster_eastus*.
1. Sur le côté gauche, choisissez **Paramètres de diagnostic**.
1. Sélectionnez votre cluster AKS, par exemple *myAKSCluster*, puis choisissez **Ajouter un paramètre de diagnostic**.
1. Entrez un nom, par exemple *myAKSClusterLogs*, puis sélectionnez l’option pour **Envoyer à Log Analytics**.
1. Sélectionnez un espace de travail existant ou créez-en un. Si vous créez un espace de travail, indiquez un nom d’espace de travail, un groupe de ressources et un emplacement.
1. Dans la liste des journaux d’activité disponibles, sélectionnez les journaux d’activité que vous souhaitez activer. Pour cet exemple, activez les journaux *kube-audit* et *kube-audit-admin*. Les journaux *kube-apiserver*, *kube-controller-manager* et *kube-scheduler* compte parmi les journaux courants. Vous pourrez réaccéder à cet emplacement et modifier les journaux d’activité collectés une fois les espaces de travail Log Analytics activés.
1. Lorsque vous avez terminé, sélectionnez **Enregistrer** pour activer la collecte des journaux d’activité sélectionnés.

## <a name="log-categories"></a>Catégories de journal

En plus des entrées écrites par Kubernetes, les journaux d’audit de votre projet comportent également des entrées provenant d’AKS.

Les journaux d’audit sont enregistrés dans trois catégories : *kube-audit*, *kube-audit-admin* et *guard*.

- La catégorie *kube-audit* contient toutes les données du journal d’audit pour chaque événement d’audit, notamment *get*, *list*, *create*, *update*, *delete*, *patch* et *post*.
- La catégorie *kube-audit-admin* est un sous-ensemble de la catégorie de journal *kube-audit*. *kube-audit-admin* réduit considérablement le nombre de journaux en excluant les événements d’audit *get* et *list* du journal.
- La catégorie *guard* correspond aux audits managés Azure AD et RBAC. Pour Azure AD managé : jeton en entrée, informations utilisateur en sortie. Pour Azure RBAC : révisions d’accès en entrée et en sortie.

## <a name="schedule-a-test-pod-on-the-aks-cluster"></a>Planifier un pod test sur le cluster AKS

Pour générer certains journaux d’activité, créez un pod dans votre cluster AKS. Vous pouvez utiliser l’exemple de manifeste YAML ci-après pour créer une instance NGINX de base. Créez un fichier nommé `nginx.yaml` dans un éditeur de votre choix, puis collez-y le contenu suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: mypod
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    ports:
    - containerPort: 80
```

Créez le pod avec la commande [kubectl create][kubectl-create] et spécifiez votre fichier YAML, comme indiqué dans l’exemple suivant :

```
$ kubectl create -f nginx.yaml

pod/nginx created
```

## <a name="view-collected-logs"></a>Afficher les journaux d’activité collectés

L’activation et l’affichage des journaux de diagnostic peuvent prendre environ 10 minutes.

> [!NOTE]
> Si vous avez besoin de toutes les données du journal d’audit à des fins de conformité ou autres, collectez-les et stockez-les dans un stockage peu onéreux, tel que Stockage Blob. Utilisez la catégorie de journal *kube-audit-admin* pour collecter et enregistrer un ensemble significatif de données de journal d’audit à des fins de surveillance et d’alerte.

Dans le portail Azure, accédez à votre cluster AKS, puis sélectionnez **Journaux** sur le côté gauche. Fermez la fenêtre *Exemples de requêtes* si elle s’affiche.

Sur le côté gauche, choisissez **Journaux d’activité**. Pour afficher le journal *kube-audit*, entrez la requête suivante dans la zone de texte :

```
AzureDiagnostics
| where Category == "kube-audit"
| project log_s
```

L’application vous renvoie alors probablement un grand nombre de journaux. Pour réduire l’étendue des résultats de la requête de façon à n’afficher que les journaux relatifs au pod NGINX créé à l’étape précédente, ajoutez une instruction *where* supplémentaire afin de rechercher *nginx*, comme indiqué dans l’exemple de requête suivant :

```
AzureDiagnostics
| where Category == "kube-audit"
| where log_s contains "nginx"
| project log_s
```

Pour afficher les journaux *kube-audit-admin*, entrez la requête suivante dans la zone de texte :

```
AzureDiagnostics
| where Category == "kube-audit-admin"
| project log_s
```

Dans cet exemple, la requête affiche tous les travaux create dans *kube-audit-admin*. Il est probable que de nombreux résultats soient renvoyés. Pour réduire l’étendue des résultats de la requête de façon à n’afficher que les journaux relatifs au pod NGINX créé à l’étape précédente, ajoutez une instruction *where* supplémentaire afin de rechercher *nginx*, comme indiqué dans l’exemple de requête suivant.

```
AzureDiagnostics
| where Category == "kube-audit-admin"
| where log_s contains "nginx"
| project log_s
```


Pour plus d’informations sur l’interrogation et le filtrage des données de journal d’activité, voir [Consulter ou analyser les données collectées avec la recherche dans les journaux d’activité de l’analytique des journaux d’activité][analyze-log-analytics].

## <a name="log-event-schema"></a>Schéma d’événement de journal

AKS enregistre les événements suivants :

* [AzureActivity][log-schema-azureactivity]
* [AzureDiagnostics][log-schema-azurediagnostics]
* [AzureMetrics][log-schema-azuremetrics]
* [ContainerImageInventory][log-schema-containerimageinventory]
* [ContainerInventory][log-schema-containerinventory]
* [ContainerLog][log-schema-containerlog]
* [ContainerNodeInventory][log-schema-containernodeinventory]
* [ContainerServiceLog][log-schema-containerservicelog]
* [Pulsation][log-schema-heartbeat]
* [InsightsMetrics][log-schema-insightsmetrics]
* [KubeEvents][log-schema-kubeevents]
* [KubeHealth][log-schema-kubehealth]
* [KubeMonAgentEvents][log-schema-kubemonagentevents]
* [KubeNodeInventory][log-schema-kubenodeinventory]
* [KubePodInventory][log-schema-kubepodinventory]
* [KubeServices][log-schema-kubeservices]
* [Perf][log-schema-perf]

## <a name="log-roles"></a>Rôles de journal

| Role                     | Description |
|--------------------------|-------------|
| *aksService*             | Nom d’affichage dans le journal d’audit de l’opération de plan de contrôle (à partir de hcpService) |
| *masterclient*           | Nom d’affichage dans le journal d’audit pour MasterClientCertificate, le certificat obtenu à partir de la commande az aks get-credentials |
| *nodeclient*             | Nom d’affichage du ClientCertificate utilisé par les nœuds d’agent |

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a expliqué comment activer et consulter les journaux d’activité des composants principaux Kubernetes de votre cluster AKS. Si vous souhaitez approfondir la supervision et la résolution des problèmes, vous pouvez également [afficher les journaux d’activité Kubelet][kubelet-logs] et [activer l’accès au nœud SSH][aks-ssh].

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create

<!-- LINKS - internal -->
[cli-quickstart]: kubernetes-walkthrough.md
[portal-quickstart]: kubernetes-walkthrough-portal.md
[log-analytics-overview]: ../azure-monitor/log-query/log-query-overview.md
[analyze-log-analytics]: ../azure-monitor/log-query/log-analytics-tutorial.md
[kubelet-logs]: kubelet-logs.md
[aks-ssh]: ssh.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[log-schema-azureactivity]: /azure/azure-monitor/reference/tables/azureactivity
[log-schema-azurediagnostics]: /azure/azure-monitor/reference/tables/azurediagnostics
[log-schema-azuremetrics]: /azure/azure-monitor/reference/tables/azuremetrics
[log-schema-containerimageinventory]: /azure/azure-monitor/reference/tables/containerimageinventory
[log-schema-containerinventory]: /azure/azure-monitor/reference/tables/containerinventory
[log-schema-containerlog]: /azure/azure-monitor/reference/tables/containerlog
[log-schema-containernodeinventory]: /azure/azure-monitor/reference/tables/containernodeinventory
[log-schema-containerservicelog]: /azure/azure-monitor/reference/tables/containerservicelog
[log-schema-heartbeat]: /azure/azure-monitor/reference/tables/heartbeat
[log-schema-insightsmetrics]: /azure/azure-monitor/reference/tables/insightsmetrics
[log-schema-kubeevents]: /azure/azure-monitor/reference/tables/kubeevents
[log-schema-kubehealth]: /azure/azure-monitor/reference/tables/kubehealth
[log-schema-kubemonagentevents]: /azure/azure-monitor/reference/tables/kubemonagentevents
[log-schema-kubenodeinventory]: /azure/azure-monitor/reference/tables/kubenodeinventory
[log-schema-kubepodinventory]: /azure/azure-monitor/reference/tables/kubepodinventory
[log-schema-kubeservices]: /azure/azure-monitor/reference/tables/kubeservices
[log-schema-perf]: /azure/azure-monitor/reference/tables/perf