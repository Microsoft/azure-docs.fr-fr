---
title: Surveiller un nouveau cluster Azure Kubernetes Service (AKS) | Microsoft Docs
description: Découvrez comment activer la surveillance d’un nouveau cluster Azure Kubernetes Service (AKS) avec Azure Monitor pour l’abonnement aux conteneurs.
ms.topic: conceptual
ms.date: 04/25/2019
ms.custom: devx-track-terraform, devx-track-azurecli
ms.openlocfilehash: 19c4a88cee8776136593b041e94dd14c7c9c28d6
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92735082"
---
# <a name="enable-monitoring-of-a-new-azure-kubernetes-service-aks-cluster"></a>Activer la surveillance d’un nouveau cluster Azure Kubernetes Service (AKS)

Cet article explique comment configurer Azure Monitor pour les conteneurs afin de surveiller un cluster Kubernetes managé hébergé sur [Azure Kubernetes Service](../../aks/index.yml) que vous préparez pour un déploiement dans votre abonnement.

Vous pouvez activer la surveillance d’un cluster AKS à l’aide d’une des méthodes prises en charge :

* Azure CLI
* Terraform

## <a name="enable-using-azure-cli"></a>Activer à l’aide d’Azure CLI

Pour activer la supervision d’un nouveau cluster AKS créé avec Azure CLI, suivez l’étape de l’article de démarrage rapide sous la section [Créer un cluster AKS](../../aks/kubernetes-walkthrough.md#create-aks-cluster).  

>[!NOTE]
>Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.74 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous aviez installé l’extension CLI aks-preview version 0.4.12 ou ultérieure, supprimez toutes les modifications que vous aviez apportées pour activer une extension en préversion. Cette extension pourrait substituer le comportement Azure CLI par défaut, étant donné que les fonctionnalités AKS en préversion ne sont pas disponibles dans le cloud Azure US Government.

## <a name="enable-using-terraform"></a>Activer à l’aide de Terraform

Si vous souhaitez [déployer un nouveau cluster AKS à l’aide de Terraform](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks), vous devez spécifier les arguments nécessaires dans le profil [pour créer un espace de travail Log Analytics](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_workspace.html), si vous n’en spécifiez pas un existant. 

>[!NOTE]
>Si vous choisissez d’utiliser Terraform, vous devez exécuter le fournisseur de gestion des ressources Azure Terraform version 1.17.0 ou ultérieure.

Pour ajouter Azure Monitor pour les conteneurs à l’espace de travail, consultez [azurerm_log_analytics_solution](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_solution.html) et renseignez le profil en incluant [**addon_profile**](https://www.terraform.io/docs/providers/azurerm/r/kubernetes_cluster.html#addon_profile) et en spécifiant **oms_agent** . 

Une fois la surveillance activée et toutes les tâches de configuration terminées correctement, vous pouvez contrôler les performances de votre cluster de deux manières :

* Directement dans le cluster AKS en sélectionnant **Intégrité** dans le volet gauche.
* En cliquant sur la vignette **Conteneurs Monitor Insights** dans la page du cluster AKS pour le cluster sélectionné. Dans Azure Monitor, dans le volet gauche, sélectionnez **Intégrité** . 

  ![Options de sélection d’Azure Monitor pour les conteneurs dans AKS](./media/container-insights-onboard/kubernetes-select-monitoring-01.png)

Une fois que vous avez activé la surveillance, 15 minutes peuvent s’écouler avant que vous puissiez voir les métriques d’intégrité du cluster. 

## <a name="verify-agent-and-solution-deployment"></a>Vérifier le déploiement de l’agent et de la solution
Avec la version *06072018* de l’agent (ou ultérieure), vous pouvez vérifier que l’agent et la solution ont été déployés correctement. Avec les versions antérieures de l’agent, vous pouvez uniquement vérifier le déploiement de l’agent.

### <a name="agent-version-06072018-or-later"></a>Agent version 06072018 ou ultérieure
Pour vérifier que l’agent a été correctement déployé, exécutez la commande suivante : 

```
kubectl get ds omsagent --namespace=kube-system
```

La sortie doit ressembler à la suivante, qui indique que l’agent a été correctement déployé :

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

Pour vérifier le déploiement de la solution, exécutez la commande suivante :

```
kubectl get deployment omsagent-rs -n=kube-system
```

La sortie doit ressembler à la suivante, qui indique que l’agent a été correctement déployé :

```
User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
omsagent   1         1         1            1            3h
```

### <a name="agent-version-earlier-than-06072018"></a>Versions de l’agent antérieures à la version 06072018

Pour vérifier que la version de l’agent Log Analytics antérieure à la version *06072018* a été correctement déployée, exécutez la commande suivante :  

```
kubectl get ds omsagent --namespace=kube-system
```

La sortie doit ressembler à la suivante, qui indique que l’agent a été correctement déployé :  

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

## <a name="view-configuration-with-cli"></a>Afficher la configuration avec l’interface de ligne de commande
Utilisez la commande `aks show` pour obtenir des détails comme l’activation ou non de la solution, le resourceID de l’espace de travail Log Analytics, et un résumé détaillé du cluster.  

```azurecli
az aks show -g <resourceGroupofAKSCluster> -n <nameofAksCluster>
```

Au bout de quelques minutes, la commande se termine et renvoie des informations formatées JSON sur la solution.  Les résultats de la commande doivent afficher le profil de module complémentaire de supervision et ressemblent à l’exemple de sortie suivant :

```
"addonProfiles": {
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/<WorkspaceSubscription>/resourceGroups/<DefaultWorkspaceRG>/providers/Microsoft.OperationalInsights/workspaces/<defaultWorkspaceName>"
      },
      "enabled": true
    }
  }
```

## <a name="next-steps"></a>Étapes suivantes

* Si vous rencontrez des problèmes en tentant d’intégrer la solution, consultez le [guide de résolution des problèmes](container-insights-troubleshoot.md)

* Une fois l’analyse activée pour collecter l’utilisation des ressources et l’intégrité de votre cluster Azure Kubernetes et charges de travail s’y exécutant, découvrez [comment utiliser](container-insights-analyze.md) Azure Monitor pour les conteneurs.

