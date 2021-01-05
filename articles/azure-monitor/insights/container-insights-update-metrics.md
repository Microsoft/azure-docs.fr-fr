---
title: Mise à jour d’Azure Monitor pour conteneurs afin d’activer les métriques | Microsoft Docs
description: Cet article décrit comment mettre à jour Azure Monitor pour conteneurs afin d’activer la fonctionnalité de métriques personnalisées prenant en charge l’exploration de métriques agrégées et la génération d’alertes sur ces dernières.
ms.topic: conceptual
ms.date: 10/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2a94f250c83fbd2779620376087a83b8851e583e
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92309449"
---
# <a name="how-to-update-azure-monitor-for-containers-to-enable-metrics"></a>Mise à jour d’Azure Monitor pour conteneurs afin d’activer les métriques

Azure Monitor pour conteneurs prend désormais en charge la collecte de métriques à partir des nœuds et pods de clusters Azure Kubernetes Service (AKS) et Kubernetes avec Azure Arc, ainsi que la consignation de ces métriques dans le magasin de métriques Azure Monitor. Cette modification est destinée à accroître la rapidité d’exécution lors de la présentation de calculs agrégés (Avg, Count, Max, Min, Sum) dans les graphiques de performances, à prendre en charge l’épinglage de ces graphiques dans les tableaux de bord du Portail Azure et à gérer les alertes de métrique.

>[!NOTE]
>Actuellement, cette fonctionnalité ne prend pas en charge les clusters Azure Red Hat OpenShift.
>

Les métriques activées dans le cadre de cette fonctionnalité sont les suivantes :

| Espace de noms de la métrique | Métrique | Description |
|------------------|--------|-------------|
| Insights.container/nodes | cpuUsageMillicores, cpuUsagePercentage, memoryRssBytes, memoryRssPercentage, memoryWorkingSetBytes, memoryWorkingSetPercentage, nodesCount, diskUsedPercentage, | Comme mesures de*nœud* métriques, elles incluent *l’hôte* en tant que dimension. Elles incluent également<br> le nom du nœud comme valeur de la dimension *host*. |
| Insights.container/pods | podCount, completedJobsCount, restartingContainerCount, oomKilledContainerCount, podReadyPercentage | Il s’agit de mesures de *pod* qui comprennent les dimensions suivantes : ControllerName, espace de noms Kubernetes, nom, phase. |
| Insights.container/containers | cpuExceededPercentage, memoryRssExceededPercentage, memoryWorkingSetExceededPercentage | |
| Insights. Container/persistentvolumes | pvUsageExceededPercentage | |

Pour prendre en charge ces nouvelles fonctionnalités, un nouvel agent en conteneur est inclus (version **microsoft/oms:ciprod05262020** pour AKS et version **microsoft/oms:ciprod09252020** pour les clusters Kubernetes avec Azure Arc). Les nouveaux déploiements d’AKS intègrent automatiquement ce changement de configuration et ces fonctionnalités. La mise à jour du cluster pour la prise en charge de cette fonctionnalité peut s’effectuer à partir du Portail Azure, d’Azure PowerShell ou de l’interface de ligne de commande Azure (Azure CLI). Avec Azure PowerShell et Azure CLI. Vous pouvez les activer par cluster ou pour tous les clusters de votre abonnement.

Les deux processus attribuent le rôle **Éditeur de métriques d’analyse** au principal de service du cluster ou à l'identité MSI affectée par l'utilisateur pour le module complémentaire de supervision afin que les données collectées par l’agent puissent être publiées sur votre ressource de cluster. L’Éditeur de métriques d’analyse est uniquement autorisé à envoyer (push) des métriques à la ressource. Il ne peut pas modifier d’état, mettre à jour la ressource ni lire aucune donnée. Pour plus d’informations sur le rôle, consultez [Monitoring Metrics Publisher role](../../role-based-access-control/built-in-roles.md#monitoring-metrics-publisher) (Rôle Éditeur de métriques d’analyse). L'exigence relative au rôle Publication des métriques de surveillance ne s'applique pas aux clusters Kubernetes avec Azure Arc.

> [!IMPORTANT]
> La mise à niveau n'est pas nécessaire pour les clusters Kubernetes avec Azure Arc car ils disposent déjà de la version minimale requise de l'agent.

## <a name="prerequisites"></a>Prérequis

Avant de mettre à jour votre cluster, vérifiez les éléments suivants :

* Les métriques personnalisées sont disponibles dans un sous-ensemble de régions Azure uniquement. La liste des régions prises en charge est présentée [ici](../platform/metrics-custom-overview.md#supported-regions).

* Vous êtes membre du rôle **[Propriétaire](../../role-based-access-control/built-in-roles.md#owner)** sur la ressource de cluster AKS pour activer la collection de métriques de performances personnalisées de nœud et de pod. Cette exigence ne s'applique pas aux clusters Kubernetes avec Azure Arc.

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.59 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="upgrade-a-cluster-from-the-azure-portal"></a>Mettre à niveau un cluster à partir du Portail Azure

Dans le cas des clusters AKS existants surveillés par Azure Monitor pour conteneurs, après avoir sélectionné un cluster pour visualiser son intégrité à partir de la vue multicluster dans Azure Monitor ou directement à partir du cluster en sélectionnant **Insights** dans le volet gauche, vous devriez voir apparaître une bannière en haut du portail.

![Bannière de mise à niveau de cluster AKS dans le Portail Azure](./media/container-insights-update-metrics/portal-banner-enable-01.png)

Cliquez sur **Activer** pour démarrer le processus de mise à niveau du cluster. Ce processus peut nécessiter plusieurs secondes. Vous pouvez suivre sa progression sous Notifications à partir du menu.

## <a name="upgrade-all-clusters-using-bash-in-azure-command-shell"></a>Mettre à niveau tous les clusters à l’aide de Bash dans l’interpréteur de commandes Azure

Pour mettre à jour tous les clusters de votre abonnement à l’aide de Bash dans l’interpréteur de commandes Azure, procédez comme suit.

1. Exécutez la commande ci-après par le biais d’Azure CLI.  Remplacez la valeur **subscriptionId** par celle qui figure sur la page **Vue d’ensemble d’AKS** du cluster AKS.

    ```azurecli
    az login
    az account set --subscription "Subscription Name"
    curl -sL https://aka.ms/ci-md-onboard-atscale | bash -s subscriptionId   
    ```

    Le changement de configuration peut nécessiter quelques minutes. Lorsqu’il est terminé, un message similaire à celui-ci s’affiche avec les résultats :

    ```azurecli
    completed role assignments for all AKS clusters in subscription: <subscriptionId>
    ```

## <a name="upgrade-per-cluster-using-azure-cli"></a>Effectuer une mise à niveau par cluster à l’aide d’Azure CLI

Pour mettre à jour un cluster spécifique de votre abonnement au moyen d’Azure CLI, procédez comme suit.

1. Exécutez la commande ci-après par le biais d’Azure CLI. Remplacez les valeurs **subscriptionId**, **resourceGroupName** et **clusterName** par celles qui figurent sur la page **Vue d’ensemble d’AKS** du cluster AKS.  La valeur **clientIdOfSPN** est renvoyée lors de l’exécution de la commande `az aks show`, comme indiqué dans l’exemple ci-dessous.

    ```azurecli
    az login
    az account set --subscription "<subscriptionName>"
    az aks show -g <resourceGroupName> -n <clusterName> 
    az role assignment create --assignee <clientIdOfSPN> --scope <clusterResourceId> --role "Monitoring Metrics Publisher" 
    ```

    Pour obtenir la valeur pour **clientIdOfSPNOrMsi**, exécutez la commande `az aks show`, comme indiqué dans l’exemple ci-dessous. Si l’objet**servicePrincipalProfile** dispose d’une valeur *clientid* valide, vous pouvez l’utiliser. Sinon, s’il est défini sur *msi*, vous devez transmettre la valeur clientid depuis `addonProfiles.omsagent.identity.clientId`.

    ```azurecli
    az login
    az account set --subscription "<subscriptionName>"
    az aks show -g <resourceGroupName> -n <clusterName> 
    az role assignment create --assignee <clientIdOfSPNOrMsi> --scope <clusterResourceId> --role "Monitoring Metrics Publisher"
    ```

## <a name="upgrade-all-clusters-using-azure-powershell"></a>Mettre à niveau tous les clusters à l’aide d’Azure PowerShell

Pour mettre à jour tous les clusters de votre abonnement à l’aide d’Azure PowerShell, procédez comme suit.

1. [Téléchargez](https://github.com/microsoft/OMS-docker/blob/ci_feature_prod/docs/aks/mdmonboarding/mdm_onboarding_atscale.ps1) le script **mdm_onboarding_atscale.ps1** et enregistrez-le dans un dossier local à partir du référentiel GitHub.
2. Exécutez la commande ci-après à l’aide d’Azure PowerShell.  Remplacez la valeur **subscriptionId** par celle qui figure sur la page **Vue d’ensemble d’AKS** du cluster AKS.

    ```powershell
    .\mdm_onboarding_atscale.ps1 subscriptionId
    ```
    Le changement de configuration peut nécessiter quelques minutes. Lorsqu’il est terminé, un message similaire à celui-ci s’affiche avec les résultats :

    ```powershell
    Completed adding role assignment for the aks clusters in subscriptionId :<subscriptionId>
    ```

## <a name="upgrade-per-cluster-using-azure-powershell"></a>Effectuer une mise à niveau par cluster à l’aide d’Azure PowerShell

Pour mettre à jour un cluster spécifique au moyen d’Azure PowerShell, procédez comme suit.

1. [Téléchargez](https://github.com/microsoft/OMS-docker/blob/ci_feature_prod/docs/aks/mdmonboarding/mdm_onboarding.ps1) le script **mdm_onboarding.ps1** et enregistrez-le dans un dossier local à partir du référentiel GitHub.

2. Exécutez la commande ci-après à l’aide d’Azure PowerShell. Remplacez les valeurs **subscriptionId**, **resourceGroupName** et **clusterName** par celles qui figurent sur la page **Vue d’ensemble d’AKS** du cluster AKS.

    ```powershell
    .\mdm_onboarding.ps1 subscriptionId <subscriptionId> resourceGroupName <resourceGroupName> clusterName <clusterName>
    ```

    Le changement de configuration peut nécessiter quelques minutes. Lorsqu’il est terminé, un message similaire à celui-ci s’affiche avec les résultats :

    ```powershell
    Successfully added Monitoring Metrics Publisher role assignment to cluster : <clusterName>
    ```

## <a name="verify-update"></a>Vérifier la mise à jour

Après avoir lancé la mise à jour à l’aide de l’une des méthodes décrites dans cet article, vous pouvez utiliser la fonctionnalité Metrics Explorer d’Azure Monitor pour vérifier que la valeur **insights** est répertoriée dans **Espace de noms de métrique**. Si tel est le cas, cela indique que vous pouvez commencer à configurer les [alertes de métrique](../platform/alerts-metric.md) ou à épingler vos graphiques dans les [tableaux de bord](../../azure-portal/azure-portal-dashboards.md).  
