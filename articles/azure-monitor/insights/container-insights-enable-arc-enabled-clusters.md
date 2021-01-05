---
title: Configurer un cluster Kubernetes compatible Azure Arc avec Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article explique comment configurer la supervision avec Azure Monitor pour conteneurs sur des clusters Kubernetes compatibles Azure Arc.
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 77b536141f0e7c6094964011719a0e536e8d33f1
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91994458"
---
# <a name="enable-monitoring-of-azure-arc-enabled-kubernetes-cluster"></a>Activer la supervision d’un cluster Kubernetes compatible Azure Arc

Azure Monitor pour les conteneurs offre une expérience d’analyse riche pour les clusters Azure Kubernetes Service (AKS) et Moteur AKS. Cet article explique comment activer la supervision de vos clusters Kubernetes hébergés en dehors d’Azure et compatibles avec Azure Arc pour obtenir une expérience de supervision similaire.

Azure Monitor pour conteneurs peut être activé pour un ou plusieurs déploiements existants de Kubernetes à l’aide d’un script PowerShell ou Bash.

## <a name="supported-configurations"></a>Configurations prises en charge

Azure Monitor pour conteneurs prend en charge la supervision de clusters Kubernetes compatibles Azure Arc (préversion) comme décrit dans l’article [Vue d’ensemble](container-insights-overview.md), à l’exception des fonctionnalités suivantes :

- Données actives (préversion)

Les éléments suivants sont officiellement pris en charge avec Azure Monitor pour conteneurs :

- Les versions de Kubernetes et de la stratégie de support sont les mêmes que celles [prises en charge par AKS](../../aks/supported-kubernetes-versions.md).

- Les runtimes de conteneur suivants sont pris en charge : Docker, Moby et les runtimes compatibles CRI tels que CRI-O et ContainerD.

- Version du système d’exploitation Linux pour les nœuds principal et Worker : Ubuntu (18.04 LTS et 16.04 LTS)

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un espace de travail Log Analytics.

    Azure Monitor pour conteneurs prend en charge un espace de travail Log Analytics dans les régions répertoriées dans [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Pour créer votre propre espace de travail, vous pouvez utiliser [Azure Resource Manager](../samples/resource-manager-workspace.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) ou le [portail Azure](../learn/quick-create-workspace.md).

- Pour activer et accéder aux fonctionnalités d’Azure Monitor pour les conteneurs, vous devez au minimum être membre du rôle Azure *Contributeur* dans l’abonnement Azure et membre du rôle [*Contributeur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) de l’espace de travail Log Analytics configuré avec Azure Monitor pour les conteneurs.

- Vous êtes membre du rôle [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) sur la ressource de cluster Azure Arc.

- Pour afficher les données de surveillance, vous êtes un membre de l’autorisation du rôle [*Lecteur Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) avec l’espace de travail Log Analytics configuré avec Azure Monitor pour les conteneurs.

- [Client HELM](https://helm.sh/docs/using_helm/) pour intégrer le graphique Azure Monitor pour conteneurs pour le cluster Kubernetes spécifié.

- Les informations de configuration du proxy et du pare-feu suivantes sont requises pour permettre à la version de l’agent Log Analytics pour Linux de communiquer avec Azure Monitor :

    |Ressource de l'agent|Ports |
    |------|---------|
    |`*.ods.opinsights.azure.com` |Port 443 |
    |`*.oms.opinsights.azure.com` |Port 443 |
    |`*.dc.services.visualstudio.com` |Port 443 |

- L’agent en conteneur requiert l’ouverture de `cAdvisor secure port: 10250` ou `unsecure port :10255` du kubelet sur tous les nœuds du cluster pour collecter les mesures de performance. Nous vous recommandons de configurer `secure port: 10250` sur le cAdvisor du kubelet s’il n’est pas déjà configuré.

- L’agent en conteneur requiert que les variables d’environnement suivantes soient spécifiées sur le conteneur pour pouvoir communiquer avec le service d’API Kubernetes au sein du cluster afin de collecter les données d’inventaire : `KUBERNETES_SERVICE_HOST` et `KUBERNETES_PORT_443_TCP_PORT`.

    >[!IMPORTANT]
    >La version minimale prise en charge de l’agent pour la supervision des clusters Kubernetes compatibles Arc est ciprod04162020 ou ultérieure.

- [PowerShell Core](/powershell/scripting/install/installing-powershell?view=powershell-6&preserve-view=true) est nécessaire si vous activez la supervision en utilisant la méthode de script PowerShell.

- [Bash version 4](https://www.gnu.org/software/bash/) est nécessaire si vous activez la supervision en utilisant la méthode de script Bash.

## <a name="identify-workspace-resource-id"></a>Identifier l’ID de ressource d’espace de travail

Pour activer la supervision de votre cluster à l’aide du script PowerShell ou bash que vous avez téléchargé précédemment et intégré à un espace de travail Log Analytics existant, effectuez les étapes suivantes pour identifier d’abord l’ID de ressource complet de votre espace de travail Log Analytics. Le paramètre `workspaceResourceId` en a besoin quand vous exécutez la commande d’activation du module complémentaire de supervision pour l’espace de travail spécifié. Si vous n’avez pas d’espace de travail à spécifier, vous pouvez passer l’étape d’inclusion du paramètre `workspaceResourceId` et laisser le script créer automatiquement un espace de travail.

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

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    Dans la sortie, recherchez le nom de l’espace de travail, puis copiez l’ID de ressource complet de cet espace de travail Log Analytics sous le champ **ID**.

## <a name="enable-monitoring-using-powershell"></a>Activer la supervision à l’aide de PowerShell

1. Téléchargez et enregistrez dans un dossier local le script qui configure votre cluster avec le module complémentaire de supervision à l’aide des commandes suivantes :

    ```powershell
    Invoke-WebRequest https://aka.ms/enable-monitoring-powershell-script -OutFile enable-monitoring.ps1
    ```

2. Configurez la variable `$azureArcClusterResourceId` en définissant les valeurs correspondantes pour `subscriptionId`, `resourceGroupName` et `clusterName` représentant l’ID de votre ressource de cluster Kubernetes compatible Azure Arc.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configurez la variable `$kubeContext` avec le **kube-context** de votre cluster en exécutant la commande `kubectl config get-contexts`. Si vous souhaitez utiliser le contexte actuel, définissez la valeur sur `""`.

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Si vous souhaitez utiliser l’espace de travail Azure Monitor Log Analytics existant, configurez la variable `$logAnalyticsWorkspaceResourceId` avec la valeur correspondante représentant l’ID de ressource de l’espace de travail. Sinon, définissez la variable sur `""` pour que le script crée un espace de travail par défaut dans le groupe de ressources par défaut de l’abonnement cluster s’il n’en existe pas déjà un dans la région. L’espace de travail créé par défaut est semblable au format de *DefaultWorkspace-\<SubscriptionID>-\<Region>* .

    ```powershell
    $logAnalyticsWorkspaceResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>"
    ```

5. Si votre cluster Kubernetes compatible Arc communique par le biais d’un serveur proxy, configurez la variable `$proxyEndpoint` avec l’URL du serveur proxy. Si le cluster ne communique pas via un serveur proxy, vous n’avez pas besoin de définir la valeur sur `""`.  Pour plus d’informations, consultez [Configurer le point de terminaison proxy](#configure-proxy-endpoint) plus loin dans cet article.

6. Exécutez la commande suivante pour activer la supervision.

    ```powershell
    .\enable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext -workspaceResourceId $logAnalyticsWorkspaceResourceId -proxyEndpoint $proxyEndpoint
    ```

Une fois que vous avez activé la surveillance, 15 minutes peuvent s’écouler avant que vous puissiez voir les métriques d’intégrité du cluster.

### <a name="using-service-principal"></a>Utilisation du principal du service
Le script *enable-monitoring.ps1* utilise la connexion interactive de l’appareil. Si vous préférez une connexion non interactive, vous pouvez utiliser un principal de service existant ou en créer un nouveau qui possède les autorisations requises, comme décrit dans [Prérequis](#prerequisites). Pour utiliser le principal du service, vous devez transmettre les paramètres $servicePrincipalClientId, $servicePrincipalClientSecret et $tenantId parameters avec les valeurs du principal de service que vous avez prévu d’utiliser pour le script *enable-monitoring.ps1*.

```powershell
$subscriptionId = "<subscription Id of the Azure Arc connected cluster resource>"
$servicePrincipal = New-AzADServicePrincipal -Role Contributor -Scope "/subscriptions/$subscriptionId"
```

L’attribution de rôle ci-dessous s’applique uniquement si vous utilisez un espace de travail Log Analytics existant dans un autre abonnement Azure que la ressource de cluster Arc K8s connectée.

```powershell
$logAnalyticsWorkspaceResourceId = "<Azure Resource Id of the Log Analytics Workspace>" # format of the Azure Log Analytics workspace should be /subscriptions/<subId>/resourcegroups/<rgName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>
New-AzRoleAssignment -RoleDefinitionName 'Log Analytics Contributor'  -ObjectId $servicePrincipal.Id -Scope  $logAnalyticsWorkspaceResourceId

$servicePrincipalClientId =  $servicePrincipal.ApplicationId.ToString()
$servicePrincipalClientSecret = [System.Net.NetworkCredential]::new("", $servicePrincipal.Secret).Password
$tenantId = (Get-AzSubscription -SubscriptionId $subscriptionId).TenantId
```

Par exemple :

```powershell
.\enable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -servicePrincipalClientId $servicePrincipalClientId -servicePrincipalClientSecret $servicePrincipalClientSecret -tenantId $tenantId -kubeContext $kubeContext -workspaceResourceId $logAnalyticsWorkspaceResourceId -proxyEndpoint $proxyEndpoint
```



## <a name="enable-using-bash-script"></a>Activer à l’aide du script bash

Effectuez les étapes suivantes pour activer la supervision à l’aide du script bash fourni.

1. Téléchargez et enregistrez dans un dossier local le script qui configure votre cluster avec le module complémentaire de supervision à l’aide des commandes suivantes :

    ```bash
    curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script
    ```

2. Configurez la variable `azureArcClusterResourceId` en définissant les valeurs correspondantes pour `subscriptionId`, `resourceGroupName` et `clusterName` représentant l’ID de votre ressource de cluster Kubernetes compatible Azure Arc.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configurez la variable `kubeContext` avec le **kube-context** de votre cluster en exécutant la commande `kubectl config get-contexts`. Si vous souhaitez utiliser le contexte actuel, définissez la valeur sur `""`.

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Si vous souhaitez utiliser l’espace de travail Azure Monitor Log Analytics existant, configurez la variable `logAnalyticsWorkspaceResourceId` avec la valeur correspondante représentant l’ID de ressource de l’espace de travail. Sinon, définissez la variable sur `""` pour que le script crée un espace de travail par défaut dans le groupe de ressources par défaut de l’abonnement cluster s’il n’en existe pas déjà un dans la région. L’espace de travail créé par défaut est semblable au format de *DefaultWorkspace-\<SubscriptionID>-\<Region>* .

    ```bash
    export logAnalyticsWorkspaceResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>"
    ```

5. Si votre cluster Kubernetes compatible Arc communique par le biais d’un serveur proxy, configurez la variable `proxyEndpoint` avec l’URL du serveur proxy. Si le cluster ne communique pas via un serveur proxy, vous n’avez pas besoin de définir la valeur sur `""`. Pour plus d’informations, consultez [Configurer le point de terminaison proxy](#configure-proxy-endpoint) plus loin dans cet article.

6. Pour activer la supervision sur votre cluster, différentes commandes sont fournies en fonction de votre scénario de déploiement.

    Exécutez la commande suivante pour activer la supervision avec les options par défaut, comme l’utilisation du kube-context actuel, créer un espace de travail Log Analytics par défaut sans spécifier de serveur proxy :

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Exécutez la commande suivante pour créer un espace de travail Log Analytics par défaut sans spécifier de serveur proxy :

    ```bash
   bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

    Exécutez la commande suivante pour utiliser un espace de travail Log Analytics existant sans spécifier de serveur proxy :

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId
    ```

    Exécutez la commande suivante pour utiliser un espace de travail Log Analytics existant et spécifier un serveur proxy :

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId --proxy $proxyEndpoint
    ```

Une fois que vous avez activé la surveillance, 15 minutes peuvent s’écouler avant que vous puissiez voir les métriques d’intégrité du cluster.

### <a name="using-service-principal"></a>Utilisation du principal du service
Le script bash *enable-monitoring.sh* utilise la connexion interactive de l’appareil. Si vous préférez une connexion non interactive, vous pouvez utiliser un principal de service existant ou en créer un nouveau qui possède les autorisations requises, comme décrit dans [Prérequis](#prerequisites). Pour utiliser le principal du service, vous devez transmettre les valeur --client-id, --client-secret et --tenant-id du principal de service que vous avez prévu d’utiliser pour le script bash *enable-monitoring.sh*.

```bash
subscriptionId="<subscription Id of the Azure Arc connected cluster resource>"
servicePrincipal=$(az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${subscriptionId}")
servicePrincipalClientId=$(echo $servicePrincipal | jq -r '.appId')
```

L’attribution de rôle ci-dessous s’applique uniquement si vous utilisez un espace de travail Log Analytics existant dans un autre abonnement Azure que la ressource de cluster Arc K8s connectée.

```bash
logAnalyticsWorkspaceResourceId="<Azure Resource Id of the Log Analytics Workspace>" # format of the Azure Log Analytics workspace should be /subscriptions/<subId>/resourcegroups/<rgName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>
az role assignment create --role 'Log Analytics Contributor' --assignee $servicePrincipalClientId --scope $logAnalyticsWorkspaceResourceId

servicePrincipalClientSecret=$(echo $servicePrincipal | jq -r '.password')
tenantId=$(echo $servicePrincipal | jq -r '.tenant')
```

Par exemple :

```bash
bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --client-id $servicePrincipalClientId --client-secret $servicePrincipalClientSecret  --tenant-id $tenantId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId --proxy $proxyEndpoint
```

## <a name="configure-proxy-endpoint"></a>Configurer le point de terminaison proxy

L’agent en conteneur pour Azure Monitor pour conteneurs vous permet de configurer un point de terminaison proxy pour lui permettre de communiquer via votre serveur proxy. La communication entre l’agent en conteneur et Azure Monitor peut s’opérer via un serveur proxy HTTP ou HTTPS, et l’authentification anonyme et l’authentification de base (nom d’utilisateur/mot de passe) sont prises en charge.

La valeur de configuration de proxy présente la syntaxe suivante : `[protocol://][user:password@]proxyhost[:port]`

> [!NOTE]
>Si votre serveur proxy n’impose pas d’authentification, vous avez quand même besoin de spécifier un pseudo nom d’utilisateur/mot de passe. Il peut s’agir de n’importe quel nom d’utilisateur ou mot de passe.

|Propriété| Description |
|--------|-------------|
|Protocol | http ou https |
|utilisateur | Nom d’utilisateur facultatif pour l’authentification du proxy |
|mot de passe | Mot de passe facultatif pour l’authentification du proxy |
|proxyhost | Adresse ou nom de domaine complet (FQDN) du serveur proxy |
|port | Numéro de port facultatif pour le serveur proxy |

Par exemple : `http://user01:password@proxy01.contoso.com:3128`

Si vous spécifiez le protocole **http**, les requêtes HTTP sont créées en utilisant une connexion sécurisée SSL/TLS. Votre serveur proxy doit prendre en charge les protocoles SSL/TLS.

### <a name="configure-using-powershell"></a>Configurer à l’aide de PowerShell

Spécifiez le nom d’utilisateur et le mot de passe, l’adresse IP ou le nom de domaine complet (FQDN) ainsi que le numéro de port du serveur proxy. Par exemple :

```powershell
$proxyEndpoint = https://<user>:<password>@<proxyhost>:<port>
```

### <a name="configure-using-bash"></a>Configurer à l’aide de bash

Spécifiez le nom d’utilisateur et le mot de passe, l’adresse IP ou le nom de domaine complet (FQDN) ainsi que le numéro de port du serveur proxy. Par exemple :

```bash
export proxyEndpoint=https://<user>:<password>@<proxyhost>:<port>
```

## <a name="next-steps"></a>Étapes suivantes

- Une fois la supervision activée pour permettre la collecte des données d’utilisation de ressources et d’intégrité de votre cluster Kubernetes compatible Arc et des charges de travail s’y exécutant, découvrez [comment utiliser](container-insights-analyze.md) Azure Monitor pour conteneurs.

- Par défaut, l’agent conteneurisé collecte les journaux de conteneurs stdout/ stderr de tous les conteneurs en cours d’exécution dans tous les espaces de noms, sauf kube-system. Pour configurer la collecte des journaux de conteneurs spécifiques d’un ou plusieurs espaces de noms particuliers, consultez [Configurer la collecte de données de l’agent pour Azure Monitor pour conteneurs](container-insights-agent-config.md) afin de configurer les paramètres de collecte de données souhaités dans votre fichier de configurations ConfigMap.

- Pour scraper et analyser des métriques Prometheus à partir de votre cluster, consultez [Configurer la capture des métriques Prometheus avec Azure Monitor pour conteneurs](container-insights-prometheus-integration.md).

- Pour savoir comment arrêter la supervision de votre cluster Kubernetes compatible Arc avec Azure Monitor pour conteneurs, consultez [Guide pratique pour arrêter la supervision de votre cluster hybride](container-insights-optout-hybrid.md#how-to-stop-monitoring-on-arc-enabled-kubernetes).