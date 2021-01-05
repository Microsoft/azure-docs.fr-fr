---
title: Activer Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article explique comment activer et configurer Azure Monitor pour conteneurs afin que vous puissiez comprendre les performances de votre conteneur et les problèmes liés aux performances identifiés.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 69022643e3346444eee95f4487eeed292c4ef139
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91994066"
---
# <a name="enable-azure-monitor-for-containers"></a>Activer Azure Monitor pour conteneurs

Cet article offre un aperçu des options disponibles pour configurer Azure Monitor pour conteneurs en vue de superviser les performances des charges de travail qui sont déployées dans un environnement Kubernetes et hébergées sur :

- [Azure Kubernetes Service (AKS)](../../aks/index.yml)  
- [Azure Red Hat OpenShift](../../openshift/intro-openshift.md) versions 3.x et 4.x  
- [Red Hat OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) version 4.x  
- Un [cluster Kubernetes avec Arc](../../azure-arc/kubernetes/overview.md)

Vous pouvez également superviser les performances des charges de travail déployées sur des clusters Kubernetes auto-gérés hébergés sur :
- Azure, à l’aide du [moteur AKS](https://github.com/Azure/aks-engine)
- [Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1910) ou localement, à l’aide du moteur AKS.

Vous pouvez activer Azure Monitor pour conteneurs pour un nouveau déploiement ou pour un ou plusieurs déploiements existants de Kubernetes à l’aide de l’une des méthodes prises en charge suivantes :

- Le portail Azure
- Azure PowerShell
- L’interface Azure CLI
- [Terraform et AKS](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous respectez les exigences suivantes :

- Vous disposez d’un espace de travail Log Analytics.

   Azure Monitor pour conteneurs prend en charge un espace de travail Log Analytics dans les régions listées dans [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor).

   Vous pouvez le créer quand vous activez la supervision de votre nouveau cluster AKS, ou vous pouvez laisser l’expérience d’intégration créer un espace de travail par défaut dans le groupe de ressources par défaut de l’abonnement de cluster AKS. 
   
   Si vous choisissez de créer l’espace de travail vous-même, vous pouvez le créer par le biais de : 
   - [Azure Resource Manager](../samples/resource-manager-workspace.md)
   - [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)
   - [Portail Azure](../learn/quick-create-workspace.md) 
   
   Pour obtenir la liste des paires de mappage prises en charge à utiliser pour l’espace de travail par défaut, consultez [Mappage des régions pour Azure Monitor pour conteneurs](container-insights-region-mapping.md).

- Vous êtes membre du groupe *Contributeur Log Analytics* pour activer la supervision des conteneurs. Pour plus d’informations sur la façon de contrôler l’accès à un espace de travail Log Analytics, consultez [Gérer les espaces de travail](../platform/manage-access.md).

- Vous êtes membre du [groupe *Propriétaire*](../../role-based-access-control/built-in-roles.md#owner) sur la ressource de cluster AKS.

   [!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

- Pour afficher les données de supervision, vous devez disposer du rôle [*Lecteur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) dans l’espace de travail Log Analytics, configuré avec Azure Monitor pour conteneurs.

- Les métriques Prometheus ne sont pas recueillies par défaut. Avant de [configurer l’agent](container-insights-prometheus-integration.md) pour recueillir les métriques, il est important de consulter la [documentation Prometheus](https://prometheus.io/) pour comprendre les données pouvant être récupérées et les méthodes prises en charge.

## <a name="supported-configurations"></a>Configurations prises en charge

Azure Monitor pour conteneurs prend officiellement en charge les configurations suivantes :

- Environnements : Azure Red Hat OpenShift, Kubernetes local et le moteur AKS sur Azure et Azure Stack. Pour plus d’informations, consultez [Moteur AKS sur Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908).
- Les versions de Kubernetes et de la stratégie de prise en charge sont les mêmes que celles [prises en charge dans Azure Kubernetes Service (AKS)](../../aks/supported-kubernetes-versions.md). 

## <a name="network-firewall-requirements"></a>Configuration requise du pare-feu réseau

Le tableau suivant répertorie les informations de configuration du proxy et du pare-feu requises pour permettre à l’agent conteneurisé de communiquer avec Azure Monitor pour conteneurs. Tout le trafic réseau provenant de l’agent est sortant vers Azure Monitor.

|Ressource de l’agent|Port |
|--------------|------|
| `*.ods.opinsights.azure.com` | 443 |
| `*.oms.opinsights.azure.com` | 443 |
| `dc.services.visualstudio.com` | 443 |
| `*.monitoring.azure.com` | 443 |
| `login.microsoftonline.com` | 443 |

Le tableau suivant répertorie les informations de configuration du proxy et du pare-feu pour Azure Chine 21Vianet :

|Ressource de l’agent|Port |Description | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.cn` | 443 | Ingestion de données |
| `*.oms.opinsights.azure.cn` | 443 | Intégration d’OMS |
| `dc.services.visualstudio.com` | 443 | Pour la télémétrie de l’agent qui utilise Azure Public Cloud Application Insights |

Le tableau suivant répertorie les informations de configuration du proxy et du pare-feu pour Azure - Gouvernement des États-Unis.

|Ressource de l’agent|Port |Description | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.us` | 443 | Ingestion de données |
| `*.oms.opinsights.azure.us` | 443 | Intégration d’OMS |
| `dc.services.visualstudio.com` | 443 | Pour la télémétrie de l’agent qui utilise Azure Public Cloud Application Insights |

## <a name="components"></a>Components

Votre capacité à superviser les performances dépend d’un agent Log Analytics conteneurisé pour Linux, spécialement développé pour Azure Monitor pour conteneurs. Cet agent spécialisé collecte les données de performances et d’événements à partir de tous les nœuds du cluster, et l’agent est automatiquement déployé puis inscrit auprès de l’espace de travail Log Analytics spécifié pendant le déploiement. 

La version d’agent est microsoft/oms:ciprod04202018 ou ultérieure. Elle est représentée par une date au format suivant : *mmjjaaaa*.

>[!NOTE]
>Avec la disponibilité générale de la prise en charge de Windows Server pour AKS, un cluster AKS avec des nœuds Windows Server dispose d’un agent de préversion installé en tant que pod daemonset sur chaque nœud de serveur Windows afin de recueillir les journaux et de les transférer à Log Analytics. Pour les métriques de performances, un nœud Linux déployé automatiquement dans le cluster dans le cadre du déploiement standard recueille et transfère les données à Azure Monitor pour le compte de tous les nœuds Windows du cluster.

Quand une nouvelle version de l’agent est disponible, celui-ci est automatiquement mis à niveau sur vos clusters Kubernetes managés hébergés sur Azure Kubernetes Service (AKS). Pour suivre les versions publiées, consultez les [annonces relatives aux versions de l’agent](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

> [!NOTE]
> Si vous avez déjà déployé un cluster AKS, vous avez activé la supervision à l’aide d’Azure CLI ou d’un modèle Azure Resource Manager fourni, comme indiqué plus loin dans cet article. Vous ne pouvez pas utiliser `kubectl` pour mettre à niveau, supprimer, redéployer ou déployer l’agent.
>
> Le modèle doit être déployé dans le même groupe de ressources que le cluster.

Pour activer Azure Monitor pour conteneurs, appliquez l’une des méthodes décrites dans le tableau suivant :

| État du déploiement | Méthode | Description |
|------------------|--------|-------------|
| Nouveau cluster Kubernetes | [Créer un cluster AKS à l’aide d’Azure CLI](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| Vous pouvez activer la supervision d’un nouveau cluster AKS que vous créez à l’aide d’Azure CLI. |
| | [Créer un cluster AKS à l’aide de Terraform](container-insights-enable-new-cluster.md#enable-using-terraform)| Vous pouvez activer la supervision d’un nouveau cluster AKS que vous créez à l’aide de l’outil open source Terraform. |
| | [Créer un cluster OpenShift à l’aide d’un modèle Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-for-a-new-cluster-using-an-azure-resource-manager-template) | Vous pouvez activer la supervision d’un cluster OpenShift que vous créez à l’aide d’un modèle Azure Resource Manager préconfiguré. |
| | [Créer un cluster OpenShift à l’aide d’Azure CLI](/cli/azure/openshift?view=azure-cli-latest#az-openshift-create) | Vous pouvez activer la supervision quand vous déployez un nouveau cluster OpenShift à l’aide d’Azure CLI. |
| Cluster Kubernetes existant | [Activer la supervision d’un cluster AKS à l’aide d’Azure CLI](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | Vous pouvez activer la supervision d’un cluster AKS déjà déployé à l’aide d’Azure CLI. |
| |[Activer pour un cluster AKS à l’aide de Terraform](container-insights-enable-existing-clusters.md#enable-using-terraform) | Vous pouvez activer la supervision d’un cluster AKS déjà déployé à l’aide de l’outil open source Terraform. |
| | [Activer pour un cluster AKS à partir d’Azure Monitor](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| Vous pouvez activer la supervision d’un ou plusieurs clusters AKS déjà déployés à partir de la page multi-cluster dans Azure Monitor. |
| | [Activer à partir du cluster AKS](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| Vous pouvez activer la surveillance directement depuis un cluster AKS dans le portail Azure. |
| | [Activer pour un cluster AKS à l’aide d’un modèle Azure Resource Manager](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| Vous pouvez activer la supervision d’un cluster AKS à l’aide d’un modèle Azure Resource Manager préconfiguré. |
| | [Activer pour le cluster Kubernetes hybride](container-insights-hybrid-setup.md) | Vous pouvez activer la supervision pour le moteur AKS qui est hébergé sur Azure Stack ou pour un cluster Kubernetes hébergé localement. |
| | [Activer pour un cluster Kubernetes avec Arc activé](container-insights-enable-arc-enabled-clusters.md). | Vous pouvez activer la supervision de vos clusters Kubernetes qui sont hébergés en dehors d’Azure et qui sont activés avec Azure Arc. |
| | [Activer pour un cluster OpenShift à l’aide d’un modèle Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-using-an-azure-resource-manager-template) | Vous pouvez activer la supervision d’un cluster OpenShift existant à l’aide d’un modèle Azure Resource Manager préconfiguré. |
| | [Activer pour un cluster OpenShift à partir d’Azure Monitor](container-insights-azure-redhat-setup.md#from-the-azure-portal) | Vous pouvez activer la supervision d’un ou plusieurs clusters OpenShift déjà déployés à partir de la page multi-cluster dans Azure Monitor. |

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez activé la supervision, vous pouvez commencer à analyser les performances de vos clusters Kubernetes hébergés sur Azure Kubernetes Service (AKS), Azure Stack ou un autre environnement. Pour savoir comment utiliser Azure Monitor pour les conteneurs, consultez [Connaître les performances des clusters Kubernetes](container-insights-analyze.md).