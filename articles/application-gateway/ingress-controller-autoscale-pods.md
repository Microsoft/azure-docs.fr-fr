---
title: Mettre à l’échelle automatiquement les pods AKS avec des métriques Azure Application Gateway
description: Cet article fournit des instructions sur la mise à l’échelle de vos blocs principaux AKS à l’aide de métriques Application Gateway et d’un adaptateur de métriques Azure Kubernetes
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: a8f015085baa8fffa6f208e9d8dd749e397c76c3
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397431"
---
# <a name="autoscale-your-aks-pods-using-application-gateway-metrics-beta"></a>Mettre à l’échelle automatiquement vos pods AKS avec des métriques Application Gateway (bêta)

À mesure que le trafic entrant augmente, il devient essentiel de faire évoluer vos applications en fonction de la demande.

Dans le didacticiel suivant, nous expliquons comment vous pouvez utiliser les métriques `AvgRequestCountPerHealthyHost` d’Application Gateway pour faire évoluer votre application. `AvgRequestCountPerHealthyHost` mesure la moyenne des requêtes envoyées à une combinaison spécifique de pool principal et de paramètres HTTP du serveur principal.

Nous allons utiliser les deux composants suivants :

* [`Azure Kubernetes Metric Adapter`](https://github.com/Azure/azure-k8s-metrics-adapter) : nous allons utiliser l’adaptateur de métriques pour exposer les métriques Application Gateway par le biais du serveur de métriques. L’adaptateur de métriques Azure Kubernetes est un projet open source sous Azure, similaire au contrôleur d’entrée Application Gateway. 
* [`Horizontal Pod Autoscaler`](../aks/concepts-scale.md#horizontal-pod-autoscaler) : nous allons nous servir de HPA pour utiliser des métriques Application Gateway et cibler un déploiement pour la mise à l’échelle.

## <a name="setting-up-azure-kubernetes-metric-adapter"></a>Configuration de l’adaptateur de métriques Azure Kubernetes

1. Nous allons tout d’abord créer un principal de service Azure AAD et lui attribuer l’accès `Monitoring Reader` via le groupe de ressources Application Gateway. 

    ```azurecli
        applicationGatewayGroupName="<application-gateway-group-id>"
        applicationGatewayGroupId=$(az group show -g $applicationGatewayGroupName -o tsv --query "id")
        az ad sp create-for-rbac -n "azure-k8s-metric-adapter-sp" --role "Monitoring Reader" --scopes applicationGatewayGroupId
    ```

1. À présent, nous allons déployer le [`Azure Kubernetes Metric Adapter`](https://github.com/Azure/azure-k8s-metrics-adapter) à l’aide du principal de service AAD créé ci-dessus.

    ```bash
    kubectl create namespace custom-metrics
    # use values from service principal created above to create secret
    kubectl create secret generic azure-k8s-metrics-adapter -n custom-metrics \
        --from-literal=azure-tenant-id=<tenantid> \
        --from-literal=azure-client-id=<clientid> \
        --from-literal=azure-client-secret=<secret>
    kubectl apply -f kubectl apply -f https://raw.githubusercontent.com/Azure/azure-k8s-metrics-adapter/master/deploy/adapter.yaml -n custom-metrics
    ```

1. Nous allons créer une ressource `ExternalMetric` avec le nom `appgw-request-count-metric`. Cette ressource indique à l’adaptateur de métriques d’exposer la métrique `AvgRequestCountPerHealthyHost` pour la ressource `myApplicationGateway` dans le groupe de ressources `myResourceGroup`. Vous pouvez utiliser le champ `filter` pour cibler un pool principal spécifique et un paramètre de serveur principal HTTP dans Application Gateway.

    ```yaml
    apiVersion: azure.com/v1alpha2
    kind: ExternalMetric
    metadata:
    name: appgw-request-count-metric
    spec:
        type: azuremonitor
        azure:
            resourceGroup: myResourceGroup # replace with your application gateway's resource group name
            resourceName: myApplicationGateway # replace with your application gateway's name
            resourceProviderNamespace: Microsoft.Network
            resourceType: applicationGateways
        metric:
            metricName: AvgRequestCountPerHealthyHost
            aggregation: Average
            filter: BackendSettingsPool eq '<backend-pool-name>~<backend-http-setting-name>' # optional
    ```

Vous pouvez maintenant effectuer une requête auprès du serveur de métriques pour voir si notre nouvelle métrique est exposée :
```bash
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1/namespaces/default/appgw-request-count-metric"
# Sample Output
# {
#   "kind": "ExternalMetricValueList",
#   "apiVersion": "external.metrics.k8s.io/v1beta1",
#   "metadata":
#     {
#       "selfLink": "/apis/external.metrics.k8s.io/v1beta1/namespaces/default/appgw-request-count-metric",
#     },
#   "items":
#     [
#       {
#         "metricName": "appgw-request-count-metric",
#         "metricLabels": null,
#         "timestamp": "2019-11-05T00:18:51Z",
#         "value": "30",
#       },
#     ],
# }
```

## <a name="using-the-new-metric-to-scale-up-the-deployment"></a>Utiliser la nouvelle métrique pour mettre à l’échelle le déploiement

Une fois que nous sommes en mesure d’exposer `appgw-request-count-metric` par le biais du serveur de métriques, nous sommes prêts à utiliser [`Horizontal Pod Autoscaler`](../aks/concepts-scale.md#horizontal-pod-autoscaler) pour faire évoluer notre déploiement cible.

Dans l’exemple suivant, nous allons cibler un exemple de déploiement `aspnet`. Nous montons en puissance les pods lorsque `appgw-request-count-metric` > 200 par pod jusqu’à un maximum de `10` pods.

Remplacez le nom de votre déploiement cible et appliquez la configuration de mise à l’échelle automatique suivante :
```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: deployment-scaler
spec:
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: aspnet # replace with your deployment's name
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: External
    external:
      metricName: appgw-request-count-metric
      targetAverageValue: 200
```

Testez votre configuration à l’aide d’un outil de test de charge comme Apache :
```bash
ab -n10000 http://<applicaiton-gateway-ip-address>/
```

## <a name="next-steps"></a>Étapes suivantes
- [**Résoudre les problèmes liés au contrôleur d’entrée**](ingress-controller-troubleshoot.md) : Résolvez les problèmes liés au contrôleur d’entrée.