---
title: Déployer des charts Helm à l’aide de GitOps sur un cluster Kubernetes compatible avec Azure Arc (préversion)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Utiliser GitOps avec Helm pour une configuration de cluster compatible avec Azure Arc (préversion)
keywords: GitOps, Kubernetes, K8s, Azure, Helm, Arc, AKS, Azure Kubernetes Service, containers
ms.openlocfilehash: eea81d458ac6631c4a023134b3198e4cdb04526e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541609"
---
# <a name="deploy-helm-charts-using-gitops-on-arc-enabled-kubernetes-cluster-preview"></a>Déployer des charts Helm à l’aide de GitOps sur un cluster Kubernetes compatible avec Azure Arc (préversion)

Helm est un outil d’empaquetage open source qui vous aide à installer et à gérer le cycle de vie d’applications Kubernetes. À l'instar de gestionnaires de package Linux tels qu'APT et Yum, Helm sert à gérer les charts Kubernetes, qui sont des packages de ressources Kubernetes préconfigurées.

Cet article vous montre comment configurer et utiliser Helm à l’aide de Kubernetes avec Azure Arc.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous disposez d’un cluster connecté Kubernetes avec Azure Arc. Si vous avez besoin d’un cluster connecté, reportez-vous au [guide de démarrage rapide Connecter un cluster](./connect-cluster.md).

## <a name="overview-of-using-gitops-and-helm-with-azure-arc-enabled-kubernetes"></a>Présentation de l’utilisation de GitOps et Helm avec Kubernetes avec Azure Arc

 L’opérateur Helm fournit une extension à Flux qui automatise les versions Chart Helm. Une version Chart est décrite par le biais d’une ressource personnalisée Kubernetes nommée HelmRelease. Flux synchronise ces ressources de Git au cluster, et l’opérateur Helm vérifie que les charts Helm sont publiés comme spécifié dans les ressources.

 L'[exemple de référentiel](https://github.com/Azure/arc-helm-demo) utilisé dans ce document est structuré comme suit :

```console
├── charts
│   └── azure-arc-sample
│       ├── Chart.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── values.yaml
└── releases
    └── app.yaml
```

Dans le référentiel Git, nous avons deux répertoires, l’un contenant un chart Helm et l’autre le fichier config des versions. Dans le répertoire `releases`, le `app.yaml` contient le fichier config HelmRelease indiqué ci-dessous :

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: azure-arc-sample
  namespace: arc-k8s-demo
spec:
  releaseName: arc-k8s-demo
  chart:
    git: https://github.com/Azure/arc-helm-demo
    ref: master
    path: charts/azure-arc-sample
  values:
    serviceName: arc-k8s-demo
```

Le fichier config de la version Helm contient les champs suivants :

- `metadata.name` est obligatoire et doit respecter les conventions d’affectation de noms de Kubernetes
- `metadata.namespace` est facultatif et détermine où la version est créée
- `spec.releaseName` est facultatif et, s’il n’est pas fourni, le nom de la version sera $namespace-$name
- `spec.chart.path` est le répertoire contenant le chart, donné par rapport à la racine de référentiel
- `spec.values` sont des personnalisations par l’utilisateur de valeurs de paramètres par défaut provenant du chart lui-même

Les options spécifiées dans le spec.values HelmRelease remplacent les options spécifiées dans values.yaml de la source du chart.

Vous pouvez en savoir plus sur le HelmRelease dans la [documentation officielle de l’opérateur Helm](https://docs.fluxcd.io/projects/helm-operator/en/stable/).

## <a name="create-a-configuration"></a>Créer une configuration

À l’aide de l’extension Azure CLI pour `k8sconfiguration`, nous allons lier notre cluster connecté à un exemple de référentiel Git. Nous donnerons à cette configuration le nom `azure-arc-sample` et déploierons l’opérateur Flux dans l’espace de noms `arc-k8s-demo`.

```console
az k8sconfiguration create --name azure-arc-sample --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name flux --operator-namespace arc-k8s-demo --operator-params='--git-readonly --git-path=releases' --enable-helm-operator --helm-operator-version='0.6.0' --helm-operator-params='--set helm.versions=v3' --repository-url https://github.com/Azure/arc-helm-demo.git --scope namespace --cluster-type connectedClusters
```

### <a name="configuration-parameters"></a>Paramètres de configuration

Pour personnaliser la création de la configuration, [découvrez les paramètres supplémentaires que vous pouvez utiliser](./use-gitops-connected-cluster.md#additional-parameters).

## <a name="validate-the-configuration"></a>Valider la configuration

À l’aide d’Azure CLI, validez la création de `sourceControlConfiguration`.

```console
az k8sconfiguration show --name azure-arc-sample --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

La ressource `sourceControlConfiguration` est mise à jour avec un état de conformité, des messages et des informations de débogage.

**Output:**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2019-12-05T05:34:41.481000",
    "message": "{\"OperatorMessage\":null,\"ClusterState\":null}",
    "messageLevel": "3"
  },
  "enableHelmOperator": "True",
  "helmOperatorProperties": {
    "chartValues": "--set helm.versions=v3",
    "chartVersion": "0.6.0"
  },
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/azure-arc-sample",
  "name": "azure-arc-sample",
  "operatorInstanceName": "flux",
  "operatorNamespace": "arc-k8s-demo",
  "operatorParams": "--git-readonly --git-path=releases",
  "operatorScope": "namespace",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "",
  "repositoryUrl": "https://github.com/Azure/arc-helm-demo.git",
  "resourceGroup": "AzureArcTest",
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

## <a name="validate-application"></a>Valider l’application

Exécutez la commande suivante et accédez à `localhost:8080` sur votre navigateur pour vérifier que l’application fonctionne.

```console
kubectl port-forward -n arc-k8s-demo svc/arc-k8s-demo 8080:8080
```

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Azure Policy pour gérer la configuration du cluster](./use-azure-policy.md)
