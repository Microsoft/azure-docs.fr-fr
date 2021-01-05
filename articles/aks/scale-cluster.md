---
title: Mettre à l’échelle un cluster Azure Kubernetes Service (AKS)
description: Apprenez à mettre le nombre de nœuds à l'échelle dans un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 09/16/2020
ms.openlocfilehash: d5686a74ffe138af51d2319c839a3a5c5887f992
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90902933"
---
# <a name="scale-the-node-count-in-an-azure-kubernetes-service-aks-cluster"></a>Mettre le nombre de nœuds à l'échelle dans un cluster Azure Kubernetes Service (AKS)

Si les besoins en ressources de vos applications évoluent, vous pouvez procéder manuellement à la mise à l'échelle d'un cluster AKS pour exécuter un nombre de nœuds différent. Si vous réduisez l’échelle, les nœuds sont soigneusement [coordonnés et purgés][kubernetes-drain] afin de limiter les perturbations pour les applications en cours d’exécution. Si vous effectuez une montée en puissance, AKS attend que les nœuds soient marqués `Ready` par le cluster Kubernetes avant que des pods soient planifiés dans ces nœuds.

## <a name="scale-the-cluster-nodes"></a>Mettre à l’échelle les nœuds de cluster

Commencez par vous procurer le *nom* de votre pool de nœuds à l’aide de la commande [az aks show][az-aks-show]. Dans l'exemple suivant, cette commande permet d’obtenir le nom du pool de nœuds du cluster *myAKSCluster* dans le groupe de ressources *myResourceGroup* :

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query agentPoolProfiles
```

L'exemple suivant montre que le *nom* est *nodepool1* :

```output
[
  {
    "count": 1,
    "maxPods": 110,
    "name": "nodepool1",
    "osDiskSizeGb": 30,
    "osType": "Linux",
    "storageProfile": "ManagedDisks",
    "vmSize": "Standard_DS2_v2"
  }
]
```

Utilisez la commande [az aks scale][az-aks-scale] pour mettre à l’échelle les nœuds du cluster. L’exemple suivant met à l’échelle un cluster nommé *myAKSCluster* vers un nœud unique. Indiquez le nom du pool de nœuds `--nodepool-name` obtenu à l'aide de la commande précédente, par exemple *nodepool1* :

```azurecli-interactive
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 1 --nodepool-name <your node pool name>
```

L'exemple suivant montre que le cluster a été mis à l'échelle avec succès sur un nœud, comme indiqué dans la section *agentPoolProfiles* :

```json
{
  "aadProfile": null,
  "addonProfiles": null,
  "agentPoolProfiles": [
    {
      "count": 1,
      "maxPods": 110,
      "name": "nodepool1",
      "osDiskSizeGb": 30,
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS2_v2",
      "vnetSubnetId": null
    }
  ],
  [...]
}
```


## <a name="scale-user-node-pools-to-0"></a>Mise à l’échelle des pools de nœuds `User` à 0

Contrairement aux pools de nœuds `System` qui requièrent toujours des nœuds en cours d’exécution, les pools de nœuds `User` vous permettent de mettre à l’échelle à 0. Pour en savoir plus sur les différences entre les pools de nœuds système et utilisateur, consultez [Pools de nœuds système et utilisateur](use-system-pools.md).

Pour mettre à l’échelle un pool d’utilisateurs à 0, vous pouvez utiliser [az aks nodepool scale][az-aks-nodepool-scale] en remplacement de la commande `az aks scale` ci-dessus et définir 0 comme nombre de nœuds.


```azurecli-interactive
az aks nodepool scale --name <your node pool name> --cluster-name myAKSCluster --resource-group myResourceGroup  --node-count 0 
```

Vous pouvez également mettre à l’échelle automatiquement les pools de nœuds `User` à 0 nœud, en affectant la valeur 0 au paramètre `--min-count` de l’[Autoscaler de cluster](cluster-autoscaler.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez mis à l’échelle un cluster AKS manuellement pour augmenter ou réduire le nombre de nœuds. Vous pouvez également utiliser l’[autoscaler de cluster][cluster-autoscaler] pour mettre à l’échelle votre cluster automatiquement.

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[cluster-autoscaler]: cluster-autoscaler.md
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-scale&preserve-view=true