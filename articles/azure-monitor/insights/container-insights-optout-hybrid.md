---
title: Arrêter la surveillance de votre cluster Kubernetes hybride | Microsoft Docs
description: Cet article décrit comment arrêter la surveillance de votre cluster Kubernetes hybride à l’aide d’Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: 2754649cd990b015162be158effa2b85aa1fe27e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90986044"
---
# <a name="how-to-stop-monitoring-your-hybrid-cluster"></a>Arrêter la surveillance de votre cluster hybride

Une fois que vous avez activé l’analyse de votre cluster Kubernetes, vous pouvez arrêter l’analyse du cluster avec Azure Monitor pour les conteneurs si vous le décidez. Cet article explique comment accomplir cette tâche pour les environnements suivants :

- Moteur AKS sur Azure et Azure Stack
- Version 4 d’OpenShift et versions ultérieures
- Kubernetes avec Azure Arc (préversion)

## <a name="how-to-stop-monitoring-using-helm"></a>Arrêt de la surveillance à l’aide de Helm

Les étapes suivantes s’appliquent à ces environnements :

- Moteur AKS sur Azure et Azure Stack
- Version 4 d’OpenShift et versions ultérieures

1. Pour identifier d’abord la version de chart Helm pour Azure Monitor pour conteneurs qui est installée sur votre cluster, exécutez la commande Helm suivante.

    ```
    helm list
    ```

    La sortie doit ressembler à ce qui suit :

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *azmon-containers-release-1* représente la version du chart Helm pour Azure Monitor pour conteneurs.

2. Pour supprimer la version du chart, exécutez la commande Helm suivante.

    `helm delete <releaseName>`

    Exemple :

    `helm delete azmon-containers-release-1`

    Cette opération supprime la version du cluster. Vous pouvez vérifier cela en exécutant la commande `helm list` :

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

Le changement de configuration peut prendre quelques minutes. Étant donné que Helm suit vos versions même après les avoir supprimées, vous pouvez auditer l’historique d’un cluster et même annuler la suppression d’une version grâce à `helm rollback`.

## <a name="how-to-stop-monitoring-on-arc-enabled-kubernetes"></a>Comment arrêter l’analyse sur les Kubernetes avec Arc activé

### <a name="using-powershell"></a>Utilisation de PowerShell

1. Téléchargez et enregistrez dans un dossier local le script qui configure votre cluster avec le module complémentaire de supervision à l’aide des commandes suivantes :

    ```powershell
    wget https://aka.ms/disable-monitoring-powershell-script -OutFile disable-monitoring.ps1
    ```

2. Configurez la variable `$azureArcClusterResourceId` en définissant les valeurs correspondantes pour `subscriptionId`, `resourceGroupName` et `clusterName` représentant l’ID de la ressource de votre ressource de cluster Kubernetes compatible avec Azure Arc.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configurez la variable `$kubeContext` avec le **kube-context** de votre cluster en exécutant la commande `kubectl config get-contexts`. Si vous souhaitez utiliser le contexte actuel, définissez la valeur sur `""`.

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Exécutez la commande suivante pour arrêter l’analyse du cluster.

    ```powershell
    .\disable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext
    ```

#### <a name="using-service-principal"></a>Utilisation du principal du service
Le script *disable-monitoring.ps1* utilise la connexion interactive de l’appareil. Si vous préférez une connexion non interactive, vous pouvez utiliser un principal de service existant ou en créer un nouveau qui possède les autorisations requises, comme décrit dans [Prérequis](container-insights-enable-arc-enabled-clusters.md#prerequisites). Pour utiliser le principal du service, vous devez transmettre les paramètres $servicePrincipalClientId, $servicePrincipalClientSecret et $tenantId parameters avec les valeurs du principal de service que vous avez prévu d’utiliser pour le script enable-monitoring.ps1.

```powershell
$subscriptionId = "<subscription Id of the Azure Arc connected cluster resource>"
$servicePrincipal = New-AzADServicePrincipal -Role Contributor -Scope "/subscriptions/$subscriptionId"

$servicePrincipalClientId =  $servicePrincipal.ApplicationId.ToString()
$servicePrincipalClientSecret = [System.Net.NetworkCredential]::new("", $servicePrincipal.Secret).Password
$tenantId = (Get-AzSubscription -SubscriptionId $subscriptionId).TenantId
```

Par exemple :

```powershell
\disable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext -servicePrincipalClientId $servicePrincipalClientId -servicePrincipalClientSecret $servicePrincipalClientSecret -tenantId $tenantId
```


### <a name="using-bash"></a>Avec Bash

1. Téléchargez et enregistrez dans un dossier local le script qui configure votre cluster avec le module complémentaire de supervision à l’aide des commandes suivantes :

    ```bash
    curl -o disable-monitoring.sh -L https://aka.ms/disable-monitoring-bash-script
    ```

2. Configurez la variable `azureArcClusterResourceId` en définissant les valeurs correspondantes pour `subscriptionId`, `resourceGroupName` et `clusterName` représentant l’ID de votre ressource de cluster Kubernetes compatible Azure Arc.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configurez la variable `kubeContext` avec le **kube-context** de votre cluster en exécutant la commande `kubectl config get-contexts`.

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Pour arrêter l’analyse de votre cluster, différentes commandes sont fournies en fonction de votre scénario de déploiement.

    Exécutez la commande suivante pour arrêter l’analyse du cluster avec le contexte actuel.

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Exécutez la commande suivante pour arrêter l’analyse du cluster en précisant un contexte

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

#### <a name="using-service-principal"></a>Utilisation du principal du service
Le script bash *disable-monitoring.sh* utilise la connexion interactive de l’appareil. Si vous préférez une connexion non interactive, vous pouvez utiliser un principal de service existant ou en créer un nouveau qui possède les autorisations requises, comme décrit dans [Prérequis](container-insights-enable-arc-enabled-clusters.md#prerequisites). Pour utiliser le principal du service, vous devez transmettre les valeur --client-id, --client-secret et --tenant-id du principal de service que vous avez prévu d’utiliser pour le script bash *enable-monitoring.sh*.

```bash
subscriptionId="<subscription Id of the Azure Arc connected cluster resource>"
servicePrincipal=$(az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${subscriptionId}")
servicePrincipalClientId=$(echo $servicePrincipal | jq -r '.appId')

servicePrincipalClientSecret=$(echo $servicePrincipal | jq -r '.password')
tenantId=$(echo $servicePrincipal | jq -r '.tenant')
```

Par exemple :

```bash
bash disable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext --client-id $servicePrincipalClientId --client-secret $servicePrincipalClientSecret  --tenant-id $tenantId
```

## <a name="next-steps"></a>Étapes suivantes

Si l’espace de travail Log Analytics a été créé uniquement pour prendre en charge la surveillance du cluster et qu’il n’est plus nécessaire, vous devez le supprimer manuellement. Si vous n’êtes pas familiarisé avec la suppression d’un espace de travail, consultez la rubrique [Supprimer un espace de travail Azure Log Analytics](../platform/delete-workspace.md).
