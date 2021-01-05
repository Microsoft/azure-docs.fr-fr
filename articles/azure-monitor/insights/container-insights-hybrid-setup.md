---
title: Configurer des clusters Kubernetes hybrides avec Azure Monitor pour les conteneurs | Microsoft Docs
description: Cet article explique comment configurer Azure Monitor pour les conteneurs afin de surveiller les clusters Kubernetes hébergés sur Azure Stack ou dans un autre environnement.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: d481af07013c0a5b4c5a381527c6f555400a2559
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890460"
---
# <a name="configure-hybrid-kubernetes-clusters-with-azure-monitor-for-containers"></a>Configurer des clusters Kubernetes hybrides avec Azure Monitor pour les conteneurs

Azure Monitor pour les conteneurs offre une expérience d’analyse riche pour Azure Kubernetes Service (AKS) et [Moteur AKS sur Azure](https://github.com/Azure/aks-engine), qui est un cluster Kubernetes automanagé hébergé sur Azure. Cet article explique comment activer la surveillance des clusters Kubernetes hébergés en dehors d’Azure et obtenir une expérience de surveillance similaire.

## <a name="supported-configurations"></a>Configurations prises en charge

Les configurations suivantes sont officiellement prises en charge avec Azure Monitor pour les conteneurs. Si vous utilisez des versions différentes de Kubernetes et du système d’exploitation, envoyez un e-mail à l’adresse askcoin@microsoft.com.

- Environnements :

    - Kubernetes en local
    - Moteur AKS sur Azure et Azure Stack. Pour plus d’informations, consultez [Moteur AKS sur Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908&preserve-view=true)
    - [OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) version 4 et ultérieure, localement ou dans d’autres environnements cloud.

- Les versions de Kubernetes et de la stratégie de support sont les mêmes que celles [prises en charge par AKS](../../aks/supported-kubernetes-versions.md).

- Les runtimes de conteneur suivants sont pris en charge : Docker, Moby et les runtimes compatibles CRI tels que CRI-O et ContainerD.

- Version du système d’exploitation Linux pour le nœud principal et les nœuds Worker : Ubuntu (18.04 LTS et 16.04 LTS) et Red Hat Enterprise Linux CoreOS 43.81.

- Contrôle d’accès pris en charge : Kubernetes RBAC et non RBAC

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un [espace de travail Log Analytics](../platform/design-logs-deployment.md).

    Azure Monitor pour conteneurs prend en charge un espace de travail Log Analytics dans les régions répertoriées dans [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Pour créer votre propre espace de travail, vous pouvez utiliser [Azure Resource Manager](../samples/resource-manager-workspace.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) ou le [portail Azure](../learn/quick-create-workspace.md).

    >[!NOTE]
    >L’analyse de plusieurs clusters avec le même nom de cluster dans le même espace de travail Log Analytics n’est pas prise en charge. Les noms de cluster doivent être uniques.
    >

- Vous êtes membre du **rôle de contributeur Log Analytics** pour activer la supervision des conteneurs. Pour plus d’informations sur la façon de contrôler l’accès à un espace de travail Log Analytics, consultez [Gérer l’accès à l’espace de travail et aux données de journal](../platform/manage-access.md).

- Pour afficher les données de supervision, vous devez disposer du rôle [*Lecteur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) dans l’espace de travail Log Analytics, configuré avec Azure Monitor pour conteneurs.

- [Client HELM](https://helm.sh/docs/using_helm/) pour intégrer le graphique Azure Monitor pour conteneurs pour le cluster Kubernetes spécifié.

- Les informations de configuration du proxy et du pare-feu suivantes sont requises pour permettre à la version de l’agent Log Analytics pour Linux de communiquer avec Azure Monitor :

    |Ressource de l'agent|Ports |
    |------|---------|
    |*.ods.opinsights.azure.com |Port 443 |
    |*.oms.opinsights.azure.com |Port 443 |
    |*.dc.services.visualstudio.com |Port 443 |

- L’agent en conteneur requiert l’ouverture de `cAdvisor secure port: 10250` ou `unsecure port :10255` du kubelet sur tous les nœuds du cluster pour collecter les mesures de performance. Nous vous recommandons de configurer `secure port: 10250` sur le cAdvisor du kubelet s’il n’est pas déjà configuré.

- L’agent en conteneur requiert que les variables d’environnement suivantes soient spécifiées sur le conteneur pour pouvoir communiquer avec le service d’API Kubernetes au sein du cluster afin de collecter les données d’inventaire : `KUBERNETES_SERVICE_HOST` et `KUBERNETES_PORT_443_TCP_PORT`.

>[!IMPORTANT]
>La version minimale prise en charge de l’agent pour la surveillance des clusters Kubernetes hybrides est ciprod10182019.

## <a name="enable-monitoring"></a>Activer la supervision

L’activation d’Azure Monitor pour les conteneurs pour le cluster Kubernetes hybride consiste à effectuer les étapes suivantes dans l’ordre.

1. Configurez votre espace de travail Log Analytics avec la solution Container Insights.   

2. Activez le graphique HELM Azure Monitor pour conteneurs avec l’espace de travail Log Analytics.

Pour plus d’informations sur les solutions de supervision dans Azure Monitor, cliquez [ici](../../azure-monitor/insights/solutions.md).

### <a name="how-to-add-the-azure-monitor-containers-solution"></a>Comment ajouter la solution Azure Monitor pour conteneurs

Vous pouvez déployer la solution avec le modèle Azure Resource Manager fourni, à l’aide de la cmdlet Azure PowerShell `New-AzResourceGroupDeployment` ou avec Azure CLI.

Si vous n’êtes pas familiarisé avec le déploiement de ressources à l’aide d’un modèle, consultez les rubriques suivantes :

- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

- [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.59 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

Cette méthode inclut deux modèles JSON. Le premier modèle spécifie la configuration permettant d’activer la surveillance, tandis que l’autre modèle contient des valeurs de paramètre que vous configurez pour spécifier les éléments suivants :

- **workspaceResourceId**  : ID de ressource complet de votre espace de travail Log Analytics.
- **workspaceRegion**  : région dans laquelle l’espace de travail est créé, également appelé **Emplacement** dans les propriétés de l’espace de travail lors de l’affichage à partir du portail Azure.

Pour identifier l’ID de ressource complet de votre espace de travail Log Analytics requis pour la valeur du paramètre `workspaceResourceId` dans le fichier de **containerSolutionParams.json** , procédez comme suit, puis exécutez l’applet de commande PowerShell ou la commande Azure CLI pour ajouter la solution.

1. Affichez la liste de tous les abonnements auxquels vous avez accès à l’aide de la commande suivante :

    ```azurecli
    az account list --all -o table
    ```

    La sortie doit ressembler à ce qui suit :

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
    ```

    Copiez la valeur de **SubscriptionId**.

2. Basculez vers l’abonnement qui héberge l’espace de travail Log Analytics à l’aide de la commande suivante :

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. L’exemple suivant affiche la liste des espaces de travail de vos abonnements au format JSON par défaut.

    ```azurecli
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    Dans la sortie, recherchez le nom de l’espace de travail, puis copiez l’ID de ressource complet de cet espace de travail Log Analytics sous le champ **ID**.

4. Copiez et collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
         }
      ]
   }
    ```

5. Enregistrez ce fichier sous le nom containerSolution.json dans un dossier local.

6. Collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "workspaceResourceId": {
          "value": "<workspaceResourceId>"
      },
      "workspaceRegion": {
        "value": "<workspaceRegion>"
      }
     }
    }
    ```

7. Modifiez les valeurs de **workspaceResourceId** à l’aide de la valeur que vous avez copiée à l’étape 3, et pour **workspaceRegion** copiez la valeur **Région** obtenue après avoir exécuté la commande Azure CLI [az monitor log-analytics workspace show](/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az-monitor-log-analytics-workspace-list&preserve-view=true).

8. Enregistrez ce fichier sous le nom containerSolutionParams.json dans un dossier local.

9. Vous êtes prêt à déployer ce modèle.

   - Pour effectuer un déploiement avec Azure PowerShell, utilisez les commandes suivantes dans le dossier qui contient le modèle :

       ```powershell
       # configure and login to the cloud of Log Analytics workspace.Specify the corresponding cloud environment of your workspace to below command.
       Connect-AzureRmAccount -Environment <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       ```

       ```powershell
       # set the context of the subscription of Log Analytics workspace
       Set-AzureRmContext -SubscriptionId <subscription Id of Log Analytics workspace>
       ```

       ```powershell
       # execute deployment command to add Container Insights solution to the specified Log Analytics workspace
       New-AzureRmResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <resource group of Log Analytics workspace> -TemplateFile .\containerSolution.json -TemplateParameterFile .\containerSolutionParams.json
       ```

       Le changement de configuration peut prendre quelques minutes. Lorsqu’il est terminé, un message similaire à celui-ci s’affiche avec les résultats :

       ```powershell
       provisioningState       : Succeeded
       ```

   - Pour effectuer un déploiement avec Azure CLI, exécutez les commandes suivantes :

       ```azurecli
       az login
       az account set --name <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       az login
       az account set --subscription "Subscription Name"
       # execute deployment command to add container insights solution to the specified Log Analytics workspace
       az deployment group create --resource-group <resource group of log analytics workspace> --name <deployment name> --template-file  ./containerSolution.json --parameters @./containerSolutionParams.json
       ```

       Le changement de configuration peut prendre quelques minutes. Lorsqu’il est terminé, un message similaire à celui-ci s’affiche avec les résultats :

       ```azurecli
       provisioningState       : Succeeded
       ```

       Une fois que vous avez activé la surveillance, 15 minutes peuvent s’écouler avant que vous puissiez voir les métriques d’intégrité du cluster.

## <a name="install-the-helm-chart"></a>Installer le graphique HELM

Dans cette section, vous allez installer l’agent en conteneur pour Azure Monitor pour les conteneurs. Avant de continuer, vous devez identifier l’ID d’espace de travail requis pour le paramètre `omsagent.secret.wsid` et la clé primaire requise pour le paramètre `omsagent.secret.key`. Vous pouvez identifier ces informations en effectuant les étapes suivantes, puis exécuter les commandes pour installer l’agent à l’aide du graphique HELM.

1. Exécutez la commande suivante pour identifier l’ID de l’espace de travail :

    `az monitor log-analytics workspace list --resource-group <resourceGroupName>`

    Dans la sortie, recherchez le nom de l’espace de travail sous le champ **nom** , puis copiez l’ID de cet espace de travail Log Analytics sous le champ **customerID**.

2. Exécutez la commande suivante pour identifier la clé primaire de l’espace de travail :

    `az monitor log-analytics workspace get-shared-keys --resource-group <resourceGroupName> --workspace-name <logAnalyticsWorkspaceName>`

    Dans la sortie, recherchez la clé primaire sous le champ **primarySharedKey** , puis copiez la valeur.

>[!NOTE]
>Les commandes suivantes s’appliquent uniquement à Helm version 2. L’utilisation du paramètre `--name` n’est pas applicable avec Helm version 3. 

>[!NOTE]
>Si votre cluster Kubernetes communique par le biais d’un serveur proxy, configurez le paramètre `omsagent.proxy` avec l’URL du serveur proxy. Si le cluster ne communique pas via un serveur proxy, vous n’avez pas besoin de spécifier ce paramètre. Pour plus d’informations, consultez [Configurer le point de terminaison proxy](#configure-proxy-endpoint) plus loin dans cet article.

3. Ajoutez le référentiel de graphiques Azure à votre liste locale en exécutant la commande suivante :

    ```
    helm repo add microsoft https://microsoft.github.io/charts/repo
    ````

4. Exécutez la commande suivante pour installer le graphique :

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<my_prod_cluster> microsoft/azuremonitor-containers
    ```

    Si l’espace de travail Log Analytics se trouve dans Azure China 21Vianet, exécutez la commande suivante :

    ```
    $ helm install --name myrelease-1 \
     --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

    Si l’espace de travail Log Analytics se trouve dans Azure US Government, exécutez la commande suivante :

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

### <a name="enable-the-helm-chart-using-the-api-model"></a>Activer le chart Helm à l’aide du modèle d’API

Vous pouvez spécifier un module complémentaire dans le fichier JSON de spécification de cluster du moteur AKS, également appelé modèle d’API. Dans ce module complémentaire, fournissez la version encodée en base64 de `WorkspaceGUID` et `WorkspaceKey` de l’espace de travail Log Analytics dans lequel sont stockées les données d’analyse collectées. Vous pouvez trouver le `WorkspaceGUID` et `WorkspaceKey` à l’aide des étapes 1 et 2 de la section précédente.

Les définitions d’API prises en charge pour le cluster Azure Stack Hub sont fournies dans cet exemple : [kubernetes-container-monitoring_existing_workspace_id_and_key.json](https://github.com/Azure/aks-engine/blob/master/examples/addons/container-monitoring/kubernetes-container-monitoring_existing_workspace_id_and_key.json). Recherchez en particulier la propriété **addons** dans **kubernetesConfig**  :

```json
"orchestratorType": "Kubernetes",
       "kubernetesConfig": {
         "addons": [
           {
             "name": "container-monitoring",
             "enabled": true,
             "config": {
               "workspaceGuid": "<Azure Log Analytics Workspace Id in Base-64 encoded>",
               "workspaceKey": "<Azure Log Analytics Workspace Key in Base-64 encoded>"
             }
           }
         ]
       }
```

## <a name="configure-agent-data-collection"></a>Configurer la collecte de données de l’agent

Avec la version de graphique 1.0.0, les paramètres de collecte des données de l’agent sont contrôlés à partir de ConfigMap. Reportez-vous à la documentation sur les paramètres de collecte de données de l’agent [ici](container-insights-agent-config.md).

Une fois que vous avez correctement déployé le graphique, vous pouvez examiner les données de votre cluster Kubernetes hybride dans Azure Monitor pour conteneurs depuis le portail Azure.  

>[!NOTE]
>La latence d’ingestion est de cinq à dix minutes entre l’agent et la validation dans l’espace de travail Azure Log Analytics. L’état du cluster affiche la valeur **Pas de données** ou **Inconnu** jusqu’à ce que toutes les données d’analyse requises soient disponibles dans Azure Monitor.

## <a name="configure-proxy-endpoint"></a>Configurer le point de terminaison proxy

À compter de la version 2.7.1 du graphique, le graphique prendra en charge la spécification du point de terminaison proxy avec le paramètre `omsagent.proxy`. Cela lui permet de communiquer via votre serveur proxy. La communication entre l’agent Azure Monitor pour les conteneurs et Azure Monitor peut être un serveur proxy HTTP ou HTTPs, et l’authentification anonyme et l’authentification de base (nom d’utilisateur/mot de passe) sont prises en charge.

La valeur de configuration de proxy a la syntaxe suivante : `[protocol://][user:password@]proxyhost[:port]`

> [!NOTE]
>Si votre serveur proxy n’impose pas d’authentification, vous avez quand même besoin de spécifier un pseudo nom d’utilisateur/mot de passe. Il peut s’agir de n’importe quel nom d’utilisateur ou mot de passe.

|Propriété| Description |
|--------|-------------|
|Protocol | http ou https |
|utilisateur | Nom d’utilisateur facultatif pour l’authentification du proxy |
|mot de passe | Mot de passe facultatif pour l’authentification du proxy |
|proxyhost | Adresse ou nom de domaine complet (FQDN) du serveur proxy |
|port | Numéro de port facultatif pour le serveur proxy |

Par exemple : `omsagent.proxy=http://user01:password@proxy01.contoso.com:8080`

Si vous spécifiez le protocole **http** , les requêtes HTTP sont créées en utilisant une connexion sécurisée SSL/TLS. Le serveur proxy doit prendre en charge les protocoles SSL/TLS.

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez une erreur lors de la tentative d’activation de la surveillance de votre cluster Kubernetes hybride, copiez le script PowerShell [TroubleshootError_nonAzureK8s.ps1](https://aka.ms/troubleshoot-non-azure-k8s) et enregistrez-le dans un dossier sur votre ordinateur. Ce script est fourni pour aider à détecter et à résoudre les problèmes rencontrés. Les problèmes qu’il est conçu de détecter et tenter de corriger sont les suivants :

- L’espace de travail Log Analytics spécifié est valide
- L’espace de travail Log Analytics est configuré avec la solution Azure Monitor pour conteneurs. Si ce n’est pas le cas, configurez l’espace de travail.
- Les pods replicaset OmsAgent sont en cours d’exécution
- Les pods daemonset OmsAgent sont en cours d’exécution
- Le service Intégrité OmsAgent est en cours d’exécution
- L’ID et la clé de l’espace de travail Log Analytics configurés sur l’agent en conteneur correspondent à l’espace de travail avec lequel Insight est configuré.
- Vérifiez que tous les nœuds Worker Linux ont l’étiquette `kubernetes.io/role=agent` pour planifier le pod rs. Si ce n’est pas le cas, ajoutez-la.
- Vérifiez que `cAdvisor secure port:10250` ou `unsecure port: 10255` est ouvert sur tous les nœuds du cluster.

Pour exécuter avec Azure PowerShell, utilisez les commandes suivantes dans le dossier qui contient le script :

```powershell
.\TroubleshootError_nonAzureK8s.ps1 - azureLogAnalyticsWorkspaceResourceId </subscriptions/<subscriptionId>/resourceGroups/<resourcegroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName> -kubeConfig <kubeConfigFile> -clusterContextInKubeconfig <clusterContext>
```

## <a name="next-steps"></a>Étapes suivantes

Une fois l’analyse activée pour collecter l’utilisation des ressources et l’intégrité de votre cluster Kubernetes hybride et des charges de travail s’y exécutant, découvrez [comment utiliser](container-insights-analyze.md) Azure Monitor pour les conteneurs.