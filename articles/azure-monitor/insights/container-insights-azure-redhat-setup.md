---
title: Configurer Azure Red Hat OpenShift v3.x avec Azure Monitor pour les conteneurs | Microsoft Docs
description: Cet article explique comment configurer la supervision d’un cluster Kubernetes avec Azure Monitor hébergé sur Azure Red Hat OpenShift versions 3 et ultérieures.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 5141ef3a96d39f16a2a9f005dd580b952046e7bf
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695662"
---
# <a name="configure-azure-red-hat-openshift-v3-with-azure-monitor-for-containers"></a>Configurer Azure Red Hat OpenShift v3.x avec Azure Monitor pour les conteneurs

>[!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service en juin 2022.
>
> À partir d’octobre 2020, vous ne pourrez plus créer de clusters 3.11.
> Les clusters 3.11 existants continueront de fonctionner jusqu’en juin 2022, mais ne seront plus pris en charge après cette date.
>
> Suivez ce guide pour [Créer un cluster Azure Red Hat OpenShift 4](../../openshift/tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, [n’hésitez pas à nous contacter](mailto:aro-feedback@microsoft.com).

Azure Monitor pour les conteneurs offre une expérience d’analyse riche pour les clusters Azure Kubernetes Service (AKS) et Moteur AKS. Cet article explique comment activer la supervision des clusters Kubernetes hébergés sur [Azure Red Hat OpenShift](../../openshift/intro-openshift.md) version 3.x et dernière évolution prise en charge de la version 3 pour obtenir une expérience de supervision similaire.

>[!NOTE]
>La prise en charge d’Azure Red Hat OpenShift est actuellement une fonctionnalité en préversion publique.
>

Azure Monitor pour conteneurs peut être activé pour un ou plusieurs déploiements d’Azure Red Hat OpenShift nouveaux ou existants, à l’aide des méthodes prises en charge suivantes :

- Pour un cluster existant à partir du portail Azure ou en utilisant un modèle Azure Resource Manager.
- Pour un nouveau cluster avec un modèle Azure Resource Manager ou lors de la création d’un cluster avec [Azure CLI](/cli/azure/openshift?view=azure-cli-latest#az-openshift-create).

## <a name="supported-and-unsupported-features"></a>Fonctionnalités prises en charge et non prises en charge

Azure Monitor pour conteneurs prend en charge la surveillance d’Azure Red Hat OpenShift comme décrit dans l’article [Vue d’ensemble](container-insights-overview.md), à l’exception des fonctionnalités suivantes :

- Données actives (préversion)
- [Collecter les métriques](container-insights-update-metrics.md) à partir des nœuds de cluster et des Pod, et les stocker dans la base de données de métriques Azure Monitor

## <a name="prerequisites"></a>Prérequis

- Un [espace de travail Log Analytics](../platform/design-logs-deployment.md).

    Azure Monitor pour conteneurs prend en charge un espace de travail Log Analytics dans les régions répertoriées dans [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Pour créer votre propre espace de travail, vous pouvez utiliser [Azure Resource Manager](../samples/resource-manager-workspace.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) ou le [portail Azure](../learn/quick-create-workspace.md).

- Pour activer et accéder aux fonctionnalités d’Azure Monitor pour les conteneurs, vous devez au minimum être membre du rôle Azure *Contributeur* dans l’abonnement Azure et membre du rôle [*Contributeur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) de l’espace de travail Log Analytics configuré avec Azure Monitor pour les conteneurs.

- Pour afficher les données de surveillance, vous êtes un membre de l’autorisation du rôle [*Lecteur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) avec l’espace de travail Log Analytics configuré avec Azure Monitor pour les conteneurs.

## <a name="identify-your-log-analytics-workspace-id"></a>Identifier votre ID d’espace de travail Log Analytics

 Pour une intégration à un espace de travail Log Analytics existant, commencez par identifier l’ID de ressource complet de votre espace de travail Log Analytics. L’ID de ressource de l’espace de travail est nécessaire pour le paramètre `workspaceResourceId` quand vous activez la supervision par le biais d’un modèle Azure Resource Manager.

1. Affichez la liste de tous les abonnements auxquels vous avez accès en exécutant la commande suivante :

    ```azurecli
    az account list --all -o table
    ```

    La sortie se présente comme suit :

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
    ```

1. Copiez la valeur de **SubscriptionId**.

1. Basculez vers l’abonnement qui héberge l’espace de travail Log Analytics en exécutant la commande suivante :

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

1. Affichez la liste des espaces de travail de vos abonnements au format JSON par défaut en exécutant la commande suivante :

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

1. Dans la sortie, recherchez le nom de l’espace de travail, puis copiez l’ID de ressource complet de cet espace de travail Log Analytics sous le champ **ID**.

## <a name="enable-for-a-new-cluster-using-an-azure-resource-manager-template"></a>Activer pour un nouveau cluster à l’aide d’un modèle Resource Manager

Effectuez les étapes suivantes pour déployer un cluster Azure Red Hat OpenShift avec la surveillance activée. Avant de continuer, consultez le didacticiel [Créer un cluster Azure Red Hat OpenShift](../../openshift/tutorial-create-cluster.md) pour comprendre les dépendances que vous devez configurer afin que votre environnement soit correctement configuré.

Cette méthode inclut deux modèles JSON. Le premier modèle spécifie la configuration permettant de déployer le cluster avec la surveillance activée, tandis que l’autre modèle contient des valeurs de paramètre que vous configurez pour spécifier les éléments suivants :

- ID de la ressource du cluster Azure Red Hat OpenShift

- Le groupe de ressources dans lequel le cluster est déployé.

- [ID de locataire Azure Active Directory](../../openshift/howto-create-tenant.md#create-a-new-azure-ad-tenant) noté après avoir effectué les étapes pour en créer un ou un déjà créé.

- [ID de l’application cliente Azure Active Directory](../../openshift/howto-aad-app-configuration.md#create-an-azure-ad-app-registration) noté après avoir effectué les étapes pour en créer un ou un déjà créé.

- [Secret client Azure Active Directory](../../openshift/howto-aad-app-configuration.md#create-a-client-secret) noté après avoir effectué les étapes pour en créer un ou un déjà créé.

- [Groupe de sécurité Azure Active Directory](../../openshift/howto-aad-app-configuration.md#create-an-azure-ad-security-group) noté après avoir effectué les étapes pour en créer un ou un déjà créé.

- ID de ressource d’un espace de travail Log Analytics existant. Pour découvrir comment obtenir ces informations, consultez [Identifier votre ID d’espace de travail Log Analytics](#identify-your-log-analytics-workspace-id).

- Le nombre de nœuds principaux pour créer un cluster.

- Le nombre de nœuds de calcul dans le profil de pool d'agents.

- Le nombre de nœuds d’infrastructure dans le profil de pool d'agents.

Si vous n’êtes pas familiarisé avec le déploiement de ressources à l’aide d’un modèle, consultez les rubriques suivantes :

- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

- [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.65 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Téléchargez et enregistrez dans un dossier local le fichier de paramètre et de modèle Azure Resource Manager pour créer un cluster avec le module complémentaire de surveillance en utilisant les commandes suivantes :

    `curl -LO https://raw.githubusercontent.com/microsoft/Docker-Provider/ci_dev/scripts/onboarding/aro/enable_monitoring_to_new_cluster/newClusterWithMonitoring.json`

    `curl -LO https://raw.githubusercontent.com/microsoft/Docker-Provider/ci_dev/scripts/onboarding/aro/enable_monitoring_to_new_cluster/newClusterWithMonitoringParam.json`

2. Connexion à Azure

    ```azurecli
    az login
    ```

    Si vous avez accès à plusieurs abonnements, exécutez `az account set -s {subscription ID}` en remplaçant `{subscription ID}` par l’abonnement que vous souhaitez utiliser.

3. Créez un groupe de ressources pour votre cluster si vous n’en avez pas déjà un. Pour obtenir une liste des régions Azure qui prennent en charge OpenShift dans Azure, consultez [Régions prises en charge](../../openshift/supported-resources.md#azure-regions).

    ```azurecli
    az group create -g <clusterResourceGroup> -l <location>
    ```

4. Modifiez le fichier de paramètre JSON **newClusterWithMonitoringParam.json** et mettez à jour les valeurs suivantes :

    - *location*
    - *clusterName*
    - *aadTenantId*
    - *aadClientId*
    - *aadClientSecret*
    - *aadCustomerAdminGroupId*
    - *workspaceResourceId*
    - *masterNodeCount*
    - *computeNodeCount*
    - *infraNodeCount*

5. L’étape suivante permet de déployer le cluster avec la surveillance activée en utilisant Azure CLI.

    ```azurecli
    az deployment group create --resource-group <ClusterResourceGroupName> --template-file ./newClusterWithMonitoring.json --parameters @./newClusterWithMonitoringParam.json
    ```

    La sortie se présente comme suit :

    ```output
    provisioningState       : Succeeded
    ```

## <a name="enable-for-an-existing-cluster"></a>Activer pour un cluster existant

Effectuez les étapes suivantes pour activer la surveillance d’un cluster Azure Red Hat OpenShift déployé dans Azure. Vous pouvez accomplir ceci depuis le portail Azure ou en utilisant les modèles fournis.

### <a name="from-the-azure-portal"></a>À partir du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le menu du Portail Azure ou dans la page Accueil, sélectionnez **Azure Monitor**. Sous la section **Insights**, sélectionnez **Conteneurs**.

3. Dans la page **Monitor - Conteneurs**, sélectionnez **Clusters non analysés**.

4. Dans la liste des clusters non surveillés, recherchez le cluster dans la liste et cliquez sur **Activer**. Vous pouvez identifier les résultats dans la liste en recherchant la valeur **ARO** sous la colonne **TYPE DE CLUSTER**.

5. Dans la page **Onboarding to Container Health and Logs** (Intégration de l’intégrité des conteneurs et aux journaux), si vous disposez d’un espace de travail Log Analytics dans le même abonnement que le cluster, sélectionnez-le dans la liste déroulante.  
    La liste présélectionne l’espace de travail par défaut et l’emplacement où le cluster est déployé dans l’abonnement.

    ![Activer la surveillance pour les clusters non surveillés](./media/container-insights-onboard/kubernetes-onboard-brownfield-01.png)

    >[!NOTE]
    >Si vous souhaitez créer un espace de travail Log Analytics pour stocker les données de supervision du cluster, suivez les instructions de [Créer un espace de travail Log Analytics](../learn/quick-create-workspace.md). Veillez à créer l’espace de travail dans le même abonnement que celui sur lequel le cluster Red Hat OpenShift est déployé.

Une fois que vous avez activé la surveillance, 15 minutes peuvent s’écouler avant que vous puissiez voir les métriques d’intégrité du cluster.

### <a name="enable-using-an-azure-resource-manager-template"></a>Activer à l’aide d’un modèle Azure Resource Manager

Cette méthode inclut deux modèles JSON. Le premier modèle spécifie la configuration permettant d’activer la surveillance, tandis que l’autre modèle contient des valeurs de paramètre que vous configurez pour spécifier les éléments suivants :

- L’ID de la ressource du cluster Azure Red Hat OpenShift.

- Le groupe de ressources dans lequel le cluster est déployé.

- Un espace de travail Log Analytics. Pour découvrir comment obtenir ces informations, consultez [Identifier votre ID d’espace de travail Log Analytics](#identify-your-log-analytics-workspace-id).

Si vous n’êtes pas familiarisé avec le déploiement de ressources à l’aide d’un modèle, consultez les rubriques suivantes :

- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

- [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.65 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Téléchargez le fichier de paramètre et de modèle pour mettre à jour votre cluster avec le module complémentaire de surveillance en utilisant les commandes suivantes :

    `curl -LO https://raw.githubusercontent.com/microsoft/Docker-Provider/ci_dev/scripts/onboarding/aro/enable_monitoring_to_existing_cluster/existingClusterOnboarding.json`

    `curl -LO https://raw.githubusercontent.com/microsoft/Docker-Provider/ci_dev/scripts/onboarding/aro/enable_monitoring_to_existing_cluster/existingClusterParam.json`

2. Connexion à Azure

    ```azurecli
    az login
    ```

    Si vous avez accès à plusieurs abonnements, exécutez `az account set -s {subscription ID}` en remplaçant `{subscription ID}` par l’abonnement que vous souhaitez utiliser.

3. Spécifiez l’abonnement du cluster Azure Red Hat OpenShift.

    ```azurecli
    az account set --subscription "Subscription Name"  
    ```

4. Exécutez la commande suivante pour identifier l’emplacement du cluster et l’ID de ressource :

    ```azurecli
    az openshift show -g <clusterResourceGroup> -n <clusterName>
    ```

5. Modifiez le fichier de paramètre JSON **existingClusterParam.json** et mettez à jour les valeurs *aroResourceId* et *aroResourceLocation*. La valeur de **workspaceResourceId** est l’ID de ressource complet de votre espace de travail Log Analytics, qui inclut le nom de l’espace de travail.

6. Pour effectuer un déploiement avec Azure CLI, exécutez les commandes suivantes :

    ```azurecli
    az deployment group create --resource-group <ClusterResourceGroupName> --template-file ./ExistingClusterOnboarding.json --parameters @./existingClusterParam.json
    ```

    La sortie se présente comme suit :

    ```output
    provisioningState       : Succeeded
    ```

## <a name="next-steps"></a>Étapes suivantes

- Une fois l’analyse activée pour collecter l’utilisation des ressources et l’intégrité de votre cluster Red Hat OpenShift et charges de travail s’y exécutant, découvrez [comment utiliser](container-insights-analyze.md) Azure Monitor pour les conteneurs.

- Par défaut, l’agent conteneurisé collecte les journaux de conteneurs stdout/ stderr de tous les conteneurs en cours d’exécution dans tous les espaces de noms, sauf kube-system. Pour configurer la collecte des journaux de conteneurs spécifiques d’un ou plusieurs espaces de noms particuliers, consultez [Configurer la collecte de données de l’agent pour Azure Monitor pour conteneurs](container-insights-agent-config.md) afin de configurer les paramètres de collecte de données souhaités dans votre fichier de configurations ConfigMap.

- Pour scraper et analyser des métriques Prometheus à partir de votre cluster, consultez [Configurer la capture des métriques Prometheus avec Azure Monitor pour conteneurs](container-insights-prometheus-integration.md).

- Pour savoir comment arrêter la surveillance de votre cluster avec Azure Monitor pour conteneurs, consultez [Comment arrêter la surveillance de votre cluster Azure Red Hat OpenShift](./container-insights-optout-openshift-v3.md).