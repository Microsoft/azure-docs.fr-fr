---
title: Connecter un cluster Kubernetes à extension Azure Arc (préversion)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Connecter un cluster Kubernetes à extension Azure Arc avec Azure Arc
keywords: Kubernetes, Arc, Azure, K8s, conteneurs
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 7f402d86ac1287753bc2deab53b24bb796644992
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97583930"
---
# <a name="connect-an-azure-arc-enabled-kubernetes-cluster-preview"></a>Connecter un cluster Kubernetes à extension Azure Arc (préversion)

Ce document décrit le processus de connexion d’un cluster Kubernetes certifié CNCF (Cloud Native Computing Foundation) comme AKS-engine sur Azure, AKS-engine sur Azure Stack Hub, GKE, EKS et VMware vSphere sur Azure Arc.

## <a name="before-you-begin"></a>Avant de commencer

Vérifiez que les exigences suivantes sont remplies :

* Cluster Kubernetes en cours d’exécution. Si vous ne disposez pas d’un cluster Kubernetes, vous pouvez utiliser l’un des guides suivants pour créer un cluster de test :
  * Créer un cluster Kubernetes à l’aide de [Kubernetes dans docker (type)](https://kind.sigs.k8s.io/)
  * Créer un cluster Kubernetes à l’aide de Docker pour [Mac](https://docs.docker.com/docker-for-mac/#kubernetes) ou [Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
* Vous avez besoin d’un fichier kubeconfig pour accéder au cluster et au rôle d’administrateur de cluster sur le cluster pour le déploiement d’agents Kubernetes activés pour Arc.
* L’utilisateur ou le principal du service utilisé avec les commandes `az login` et `az connectedk8s connect` doit avoir les autorisations en lecture et en écriture sur le type de ressource « Microsoft. Kubernetes/connectedclusters ». Le rôle « Intégration d’Azure Arc pour cluster Kubernetes » disposant de ces autorisations peut être utilisé pour les attributions de rôles sur l’utilisateur ou le principal du service.
* Helm 3 est requis pour l’intégration du cluster à l’aide de l’extension connectedk8s. [Installez la dernière version de Helm 3](https://helm.sh/docs/intro/install) pour répondre à cette exigence.
* Pour installer les extensions CLI de Kubernetes activées d’Azure Arc, vous avez besoin de la version 2.3+ d’Azure CLI. [Installez Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) ou mettez-la à jour vers la dernière version pour vous assurer d’avoir Azure CLI version 2.3 +.
* Installez les extensions Kubernetes CLI compatibles avec Arc :
  
  Installez l’extension `connectedk8s` qui vous aide à connecter des clusters Kubernetes à Azure :
  
  ```console
  az extension add --name connectedk8s
  ```
  
  Installez l’extension `k8sconfiguration`.
  
  ```console
  az extension add --name k8sconfiguration
  ```
  
  Si vous souhaitez mettre à jour ces extensions ultérieurement, exécutez les commandes suivantes :
  
  ```console
  az extension update --name connectedk8s
  az extension update --name k8sconfiguration
  ```

## <a name="supported-regions"></a>Régions prises en charge

* USA Est
* Europe Ouest

## <a name="network-requirements"></a>Configuration requise pour le réseau

Les agents Azure Arc requièrent les protocoles/ports/URL sortantes suivants pour fonctionner.

* TCP sur le port 443 --> `https://:443`
* TCP sur le port 9418 --> `git://:9418`

| Point de terminaison (DNS)                                                                                               | Description                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `https://management.azure.com`                                                                                 | Requis pour que l’agent se connecte à Azure et inscrive le cluster.                                                        |
| `https://eastus.dp.kubernetesconfiguration.azure.com`, `https://westeurope.dp.kubernetesconfiguration.azure.com` | Point de terminaison du plan de données pour que l’agent transmette les informations de configuration d’état et de récupération.                                      |
| `https://login.microsoftonline.com`                                                                            | Requis pour extraire et mettre à jour des jetons Azure Resource. Manager                                                                                    |
| `https://mcr.microsoft.com`                                                                            | Requis pour extraire des images de conteneur pour les agents Azure Arc.                                                                  |
| `https://eus.his.arc.azure.com`, `https://weu.his.arc.azure.com`                                                                            |  Requis pour extraire les certificats d'identité managée attribués par le système                                                                  |

## <a name="register-the-two-providers-for-azure-arc-enabled-kubernetes"></a>Inscrire les deux fournisseurs pour Kubernetes à extension Azure Arc :

```console
az provider register --namespace Microsoft.Kubernetes

az provider register --namespace Microsoft.KubernetesConfiguration
```

L’inscription est un processus asynchrone. L’inscription peut prendre environ 10 minutes. Vous pouvez surveiller le processus d’inscription à l’aide des commandes suivantes :

```console
az provider show -n Microsoft.Kubernetes -o table
```

```console
az provider show -n Microsoft.KubernetesConfiguration -o table
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Utilisez un groupe de ressources pour stocker les métadonnées pour votre cluster.

Tout d’abord, créez un groupe de ressources pour contenir la ressource de cluster connectée.

```console
az group create --name AzureArcTest -l EastUS -o table
```

**Output:**

```console
Location    Name
----------  ------------
eastus      AzureArcTest
```

## <a name="connect-a-cluster"></a>Connecter un cluster

Nous allons ensuite connecter notre cluster Kubernetes à Azure. Le flux de travail pour `az connectedk8s connect` est le suivant :

1. Vérifiez la connectivité à votre cluster Kubernetes : via `KUBECONFIG`, `~/.kube/config` ou `--kube-config`
1. Déployez des agents Azure Arc pour Kubernetes en utilisant Helm 3, dans l’espace de noms `azure-arc`

```console
az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
```

**Output:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Helm release deployment succeeded

{
  "aadProfile": {
    "clientAppId": "",
    "serverAppId": "",
    "tenantId": ""
  },
  "agentPublicKeyCertificate": "...",
  "agentVersion": "0.1.0",
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
  "identity": {
    "principalId": null,
    "tenantId": null,
    "type": "None"
  },
  "kubernetesVersion": "v1.15.0",
  "location": "eastus",
  "name": "AzureArcTest1",
  "resourceGroup": "AzureArcTest",
  "tags": {},
  "totalNodeCount": 1,
  "type": "Microsoft.Kubernetes/connectedClusters"
}
```

## <a name="verify-connected-cluster"></a>Vérifier un cluster connecté

Répertoriez vos clusters connectés :

```console
az connectedk8s list -g AzureArcTest -o table
```

**Output:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

Vous pouvez également afficher cette ressource dans le [Portail Azure](https://portal.azure.com/). Après avoir ouvert le portail dans votre navigateur, recherchez dans le groupe de ressources et la ressource Kubernetes activée d’Azure Arc basés sur le nom de ressource et le nom de du groupe de ressources utilisés plus tôt dans la commande `az connectedk8s connect`.

> [!NOTE]
> Après l’intégration du cluster, il faut environ 5 à 10 minutes pour que les métadonnées du cluster (version du cluster, version de l’agent, nombre de nœuds) soient visibles sur la page Vue d’ensemble de la ressource Kubernetes avec Azure Arc activé dans le portail Azure.

## <a name="connect-using-an-outbound-proxy-server"></a>Se connecter à l’aide d’un serveur proxy sortant

Si votre cluster se trouve derrière un serveur proxy sortant, Azure CLI et les agents Kubernetes activés pour Arc doivent acheminer leurs demandes via le serveur proxy sortant. La configuration suivante permet ce qui suit :

1. Vérifiez la version de l’extension `connectedk8s` installée sur votre ordinateur en exécutant la commande suivante :

    ```console
    az -v
    ```

    Vous devez disposer de l’extension `connectedk8s` version >= 0.2.5 pour configurer des agents avec un proxy sortant. Si vous disposez d’une version < 0.2.3 sur votre ordinateur, suivez les [étapes de mise à jour](#before-you-begin) pour obtenir la dernière version de l’extension sur votre ordinateur.

2. Définissez les variables d’environnement nécessaires pour Azure CLI afin d’utiliser le serveur proxy sortant :

    * Si vous utilisez bash, exécutez la commande suivante avec les valeurs appropriées :

        ```bash
        export HTTP_PROXY=<proxy-server-ip-address>:<port>
        export HTTPS_PROXY=<proxy-server-ip-address>:<port>
        export NO_PROXY=<cluster-apiserver-ip-address>:<port>
        ```

    * Si vous utilisez PowerShell, exécutez la commande suivante avec les valeurs appropriées :

        ```powershell
        $Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"
        ```

3. Exécutez la commande connect avec les paramètres de proxy spécifiés :

    ```console
    az connectedk8s connect -n <cluster-name> -g <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --proxy-cert <path-to-cert-file>
    ```

> [!NOTE]
> 1. La spécification de excludedCIDR sous --proxy-skip-range est importante pour garantir que la communication dans le cluster n’est pas interrompue pour les agents.
> 2. Tandis que les valeurs --proxy-http, --proxy-https et --proxy-skip-range sont attendues pour la plupart des environnements de proxy sortants, --proxy-cert est requis uniquement s’il existe des certificats de confiance provenant du proxy qui doivent être injectés dans le magasin de certificats de confiance des pods d’agent.
> 3. La spécification de proxy ci-dessus est actuellement appliquée uniquement pour les agents Arc et non pour les pods de flux utilisés dans sourceControlConfiguration. L’équipe Kubernetes avec Arc travaille activement sur cette fonctionnalité, qui sera bientôt disponible.

## <a name="azure-arc-agents-for-kubernetes"></a>Agents Azure Arc pour Kubernetes

Un Kubernetes à extension Azure Arc déploie quelques opérateurs dans l’espace de noms `azure-arc`. Vous pouvez afficher ces déploiements et pods ici :

```console
kubectl -n azure-arc get deployments,pods
```

**Output:**

```console
NAME                                        READY      UP-TO-DATE  AVAILABLE  AGE
deployment.apps/cluster-metadata-operator     1/1             1        1      16h
deployment.apps/clusteridentityoperator       1/1             1        1      16h
deployment.apps/config-agent                  1/1             1        1      16h
deployment.apps/controller-manager            1/1             1        1      16h
deployment.apps/flux-logs-agent               1/1             1        1      16h
deployment.apps/metrics-agent                 1/1             1        1      16h
deployment.apps/resource-sync-agent           1/1             1        1      16h

NAME                                           READY    STATUS   RESTART AGE
pod/cluster-metadata-operator-7fb54d9986-g785b  2/2     Running  0       16h
pod/clusteridentityoperator-6d6678ffd4-tx8hr    3/3     Running  0       16h
pod/config-agent-544c4669f9-4th92               3/3     Running  0       16h
pod/controller-manager-fddf5c766-ftd96          3/3     Running  0       16h
pod/flux-logs-agent-7c489f57f4-mwqqv            2/2     Running  0       16h
pod/metrics-agent-58b765c8db-n5l7k              2/2     Running  0       16h
pod/resource-sync-agent-5cf85976c7-522p5        3/3     Running  0       16h
```

Un Kubernetes à extension Azure Arc se compose de quelques agents (opérateurs) qui s’exécutent dans votre cluster déployé dans l’espace de noms `azure-arc`.

* `deployment.apps/config-agent` : surveille le cluster connecté pour les ressources de configuration de contrôle de code source appliquées au cluster, et met à jour l’état de conformité.
* `deployment.apps/controller-manager` : opérateur d’opérateurs orchestrant les interactions entre les composants Azure Arc.
* `deployment.apps/metrics-agent` : collecte les métriques d’autres agents Arc pour s’assurer que ces agents offrent des performances optimales.
* `deployment.apps/cluster-metadata-operator` : collecte des métadonnées de cluster : version du cluster, nombre de nœuds et version de l’agent Azure Arc
* `deployment.apps/resource-sync-agent` : synchronise les métadonnées de cluster précitées dans Azure.
* `deployment.apps/clusteridentityoperator`: Kubernetes avec Azure Arc prend actuellement en charge l’identité attribuée par le système. clusteridentityoperator gère le certificat d’identité de service managé (MSI) utilisé par d’autres agents pour la communication avec Azure.
* `deployment.apps/flux-logs-agent` : collecte des journaux des opérateurs de flux déployés dans le cadre de la configuration du contrôle de code source.

## <a name="delete-a-connected-cluster"></a>Supprimer un cluster connecté

Vous pouvez supprimer une ressource `Microsoft.Kubernetes/connectedcluster` à l’aide d’Azure CLI ou du portail Azure.


* **Suppression avec Azure CLI** : La commande Azure CLI suivante peut être utilisée pour supprimer une ressource Kubernetes activée d’Azure Arc.
  ```console
  az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
  ```
  Cette commande supprime la `Microsoft.Kubernetes/connectedCluster`ressource et toute ressource qui y est associée`sourcecontrolconfiguration` dans Azure. Azure CLI utilise une désinstallation de Helm pour supprimer les agents du cluster en cours d’exécution.

* **Suppression dans le portail Azure** : La suppression de la ressource Kubernetes activée d’Azure Arc sur le portail Azure permet de supprimer la ressource `Microsoft.Kubernetes/connectedcluster` et toutes les ressources qui y sont associées`sourcecontrolconfiguration` dans Azure, mais cela ne supprime pas les agents en cours d’exécution sur le cluster. Pour supprimer ces agents, vous devez exécuter la commande suivante.

  ```console
  az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
  ```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser GitOps dans un cluster connecté](./use-gitops-connected-cluster.md)
* [Utiliser Azure Policy pour gérer la configuration du cluster](./use-azure-policy.md)
