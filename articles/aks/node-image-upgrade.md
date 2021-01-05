---
title: Mettre à niveau les images de nœud Azure Kubernetes service (AKS)
description: Découvrez comment mettre à niveau les images sur des nœuds de cluster AKS et des pools de nœuds.
ms.service: container-service
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: jpalma
ms.openlocfilehash: 83d7d48922806334e2b49494fe0ef1d15e1a7a6a
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531477"
---
# <a name="azure-kubernetes-service-aks-node-image-upgrade"></a>Mise à niveau des images de nœud Azure Kubernetes service (AKS)

AKS prend en charge la mise à niveau des images sur un nœud afin que vous soyez à jour des dernières mises à jour du système d’exploitation et du runtime. AKS fournit une nouvelle image par semaine avec les dernières mises à jour. Il est donc préférable de régulièrement mettre à niveau les images de votre nœud pour bénéficier des dernières fonctionnalités, notamment les correctifs Linux ou Windows. Cet article explique comment mettre à niveau des images de nœud de cluster AKS et comment mettre à jour des images de pool de nœuds sans mettre à niveau la version de Kubernetes.

Pour plus d’informations sur les images les plus récentes fournies par AKS, consultez les [notes de publication AKS](https://github.com/Azure/AKS/releases).

Pour plus d’informations sur la mise à niveau de la version Kubernetes pour votre cluster, consultez [Mettre à niveau un cluster AKS][upgrade-cluster].

> [!NOTE]
> Le cluster AKS doit utiliser des groupes de machines virtuelles identiques pour les nœuds.

## <a name="check-if-your-node-pool-is-on-the-latest-node-image"></a>Vérifier si votre pool de nœuds se trouve sur l’image de nœud la plus récente

Vous pouvez identifier la dernière version de l’image de nœud disponible pour votre pool de nœuds à l’aide de la commande suivante : 

```azurecli
az aks nodepool get-upgrades \
    --nodepool-name mynodepool \
    --cluster-name myAKSCluster \
    --resource-group myResourceGroup
```

Dans la sortie, vous pouvez voir l’image `latestNodeImageVersion`, comme dans l’exemple ci-dessous :

```output
{
  "id": "/subscriptions/XXXX-XXX-XXX-XXX-XXXXX/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/nodepool1/upgradeProfiles/default",
  "kubernetesVersion": "1.17.11",
  "latestNodeImageVersion": "AKSUbuntu-1604-2020.10.28",
  "name": "default",
  "osType": "Linux",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.ContainerService/managedClusters/agentPools/upgradeProfiles",
  "upgrades": null
}
```

Par conséquent, pour `nodepool1`, la dernière image de nœud disponible est `AKSUbuntu-1604-2020.10.28`. Vous pouvez maintenant la comparer à la version actuelle de l’image de nœud en cours d’utilisation par votre pool de nœuds en exécutant :

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --query nodeImageVersion
```

Voici un exemple de sortie :

```output
"AKSUbuntu-1604-2020.10.08"
```

Ainsi, dans cet exemple, vous pouvez effectuer la mise à niveau de la version actuelle de l’image `AKSUbuntu-1604-2020.10.08` vers la dernière version `AKSUbuntu-1604-2020.10.28`. 

## <a name="upgrade-all-nodes-in-all-node-pools"></a>Mettre à niveau tous les nœuds dans tous les pools de nœuds

La mise à niveau de l’image de nœud s’effectue avec `az aks upgrade`. Pour mettre à niveau l’image de nœud, utilisez la commande suivante :

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-image-only
```

Pendant la mise à niveau, vérifiez l’état des images de nœud à l’aide de la commande `kubectl` suivante pour récupérer les étiquettes et filtrer les informations sur l’image de nœud actuelle :

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Une fois la mise à niveau terminée, utilisez `az aks show` pour obtenir les détails du pool de nœuds mis à jour. L’image de nœud actuelle est affichée dans la propriété `nodeImageVersion`.

```azurecli
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster
```

## <a name="upgrade-a-specific-node-pool"></a>Mettre à niveau un pool de nœuds spécifique

La mise à niveau de l’image sur un pool de nœuds est semblable à la mise à niveau de l’image sur un cluster.

Pour mettre à jour l’image du système d’exploitation du pool de nœuds sans effectuer de mise à niveau du cluster Kubernetes, utilisez l’option `--node-image-only` dans l’exemple suivant :

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-image-only
```

Pendant la mise à niveau, vérifiez l’état des images de nœud à l’aide de la commande `kubectl` suivante pour récupérer les étiquettes et filtrer les informations sur l’image de nœud actuelle :

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Une fois la mise à niveau terminée, utilisez `az aks nodepool show` pour obtenir les détails du pool de nœuds mis à jour. L’image de nœud actuelle est affichée dans la propriété `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="upgrade-node-images-with-node-surge"></a>Mettre à niveau les images de nœud à l’aide de l’augmentation du nombre de nœuds

Pour accélérer le processus de mise à niveau des images de nœud, vous pouvez mettre à niveau vos images de nœud à l’aide d’une valeur personnalisable d’augmentation du nombre de nœuds. Par défaut, AKS utilise un nœud supplémentaire pour configurer les mises à niveau.

Si vous souhaitez augmenter la vitesse des mises à niveau, utilisez la valeur `--max-surge` pour configurer le nombre de nœuds à utiliser pour les mises à niveau afin qu’elles se terminent plus rapidement. Pour en savoir plus sur les compromis des différents paramètres `--max-surge`, consultez [Personnaliser la mise à niveau avec l’augmentation du nombre de nœuds][max-surge].

La commande suivante définit la valeur max-surge pour effectuer la mise à niveau d’une image de nœud :

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --max-surge 33% \
    --node-image-only \
    --no-wait
```

Pendant la mise à niveau, vérifiez l’état des images de nœud à l’aide de la commande `kubectl` suivante pour récupérer les étiquettes et filtrer les informations sur l’image de nœud actuelle :

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Utilisez `az aks nodepool show` pour récupérer les détails du pool de nœuds mis à jour. L’image de nœud actuelle est affichée dans la propriété `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les images de nœud les plus récentes, consultez les [notes de publication AKS](https://github.com/Azure/AKS/releases).
- Apprenez à mettre à niveau la version de Kubernetes avec la section [Mettre à niveau un cluster AKS][upgrade-cluster].
- [Appliquer automatiquement des mises à niveau de pools de clusters et de nœuds avec GitHub Actions][github-schedule]
- En savoir plus sur les pools de nœuds multiples et sur la mise à niveau des pools de nœuds avec la section [Créer et gérer les pools de nœuds multiples][use-multiple-node-pools].

<!-- LINKS - internal -->
[upgrade-cluster]: upgrade-cluster.md
[github-schedule]: node-upgrade-github-actions.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
