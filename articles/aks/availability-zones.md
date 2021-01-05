---
title: Utiliser des zones de disponibilité dans Azure Kubernetes Service (AKS)
description: Découvrez comment créer un cluster qui distribue les nœuds entre les zones de disponibilité dans Azure Kubernetes Service (AKS)
services: container-service
ms.custom: fasttrack-edit, references_regions, devx-track-azurecli
ms.topic: article
ms.date: 09/04/2020
ms.openlocfilehash: 15f66e836a2900349007fb5068a172b89f39d4de
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352794"
---
# <a name="create-an-azure-kubernetes-service-aks-cluster-that-uses-availability-zones"></a>Créer un cluster Azure Kubernetes Service (AKS) qui utilise des zones de disponibilité

Un cluster AKS (Azure Kubernetes Service) répartit des ressources telles que des nœuds et le stockage entre des sections logiques de l’infrastructure Azure sous-jacente. Ce modèle de déploiement, lors de l’utilisation de zones de disponibilité, garantit que les nœuds d’une zone de disponibilité donnée sont physiquement séparés de ceux définis dans une autre zone de disponibilité. Les clusters AKS déployés avec plusieurs zones de disponibilité configurées sur un cluster offrent un niveau plus élevé de disponibilité pour se protéger contre une défaillance matérielle ou un événement de maintenance planifiée.

En définissant des pools de nœuds dans un cluster de façon à ce qu’ils s’étendent sur plusieurs zones, les nœuds d’un pool de nœuds donné peuvent continuer à fonctionner même si une zone tombe en panne. Vos applications peuvent continuer à être disponibles même en cas de défaillance physique dans un centre de donnée si elles sont orchestrées de manière à tolérer la défaillance d’un sous-ensemble de nœuds.

Cet article vous explique comment créer un cluster AKS et répartir les composants de nœud entre des zones de disponibilité.

## <a name="before-you-begin"></a>Avant de commencer

La version 2.0.76 d’Azure CLI (ou ultérieure) doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="limitations-and-region-availability"></a>Limitations et disponibilité de la région

Les clusters AKS peuvent actuellement être créés en utilisant des zones de disponibilité dans les régions suivantes :

* Australie Est
* Centre du Canada
* USA Centre
* USA Est 
* USA Est 2
* France Centre
* Japon Est
* Europe Nord
* Asie Sud-Est
* Sud du Royaume-Uni
* Europe Ouest
* USA Ouest 2

Les limitations suivantes s'appliquent lorsque vous créez un cluster AKS à l'aide de zones de disponibilité :

* Vous ne pouvez définir des zones de disponibilité que lorsque le cluster ou le pool de nœuds est créé.
* Les paramètres de la zone de disponibilité ne peuvent pas être mis à jour une fois le cluster créé. Vous ne pouvez pas non plus mettre à jour un cluster de zones de non-disponibilité existant pour utiliser des zones de disponibilité.
* La taille de nœud (référence SKU de la machine virtuelle) choisie doit être disponible dans toutes les zones de disponibilité sélectionnées.
* Les clusters avec des zones de disponibilité activées nécessitent l'utilisation d'équilibreurs de charge Standard Azure pour la distribution entre les zones. Ce type d’équilibreur de charge ne peut être défini qu’au moment de la création du cluster. Pour plus d'informations et connaître les limites de l’équilibreur de charge Standard, consultez [Limites de la référence SKU Standard de l’équilibreur de charge Azure][standard-lb-limitations].

### <a name="azure-disks-limitations"></a>Limitations des disques Azure

Les volumes qui utilisent des disques managés Azure ne constituent pas actuellement des ressources redondantes interzones. Les volumes ne peuvent pas être attachés entre les zones et doivent être colocalisés dans la même zone qu’un nœud donné hébergeant un pod cible.

Kubernetes prend en charge les zones de disponibilité Azure depuis la version 1.12. Vous pouvez déployer un objet PersistentVolumeClaim référençant un disque managé Azure dans un cluster AKS à plusieurs zones et [Kubernetes s’occupera de la planification](https://kubernetes.io/docs/setup/best-practices/multiple-zones/#storage-access-for-zones) de tout Pod qui revendique ce PVC dans la zone de disponibilité adéquate.

## <a name="overview-of-availability-zones-for-aks-clusters"></a>Vue d’ensemble des zones de disponibilité pour les clusters AKS

Les zones de disponibilité constituent une offre à haute disponibilité qui protège vos applications et données des pannes des centres de données. Les zones sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Pour garantir la résilience, il existe toujours plus d’une zone dans toutes les régions activées. La séparation physique des zones de disponibilité dans une région protège les applications et les données des défaillances dans le centre de données.

Pour plus d’informations, consultez [Que sont les zones de disponibilité dans Azure ?][az-overview].

Les clusters AKS déployés à l'aide de zones de disponibilité peuvent répartir les nœuds sur plusieurs zones au sein d'une même région. Par exemple, un cluster dans la région  *USA Est 2*  peut créer des nœuds dans les trois zones de disponibilité de *USA Est 2*. Cette distribution des ressources du cluster AKS améliore la disponibilité du cluster car elles résistent aux défaillances d'une zone spécifique.

![Distribution des nœuds AKS entre les zones de disponibilité](media/availability-zones/aks-availability-zones.png)

Si une zone devient indisponible, vos applications continuent à s’exécuter si le cluster est réparti sur plusieurs zones.

## <a name="create-an-aks-cluster-across-availability-zones"></a>Créer un cluster AKS sur plusieurs zones de disponibilité

Lorsque vous créez un cluster à l'aide de la commande [az aks create][az-aks-create], le paramètre `--zones` définit dans quelles zones les nœuds d'agent sont déployés. Les composants du plan de contrôle, tels que etcd ou l’API, sont répartis sur les zones disponibles dans la région si vous définissez le paramètre `--zones` au moment de la création du cluster. Les zones spécifiques sur lesquelles sont répartis les composants du plan de contrôle sont indépendantes des zones explicites sélectionnées pour le pool de nœuds initial.

Si vous ne définissez aucune zone pour le pool d’agents par défaut lorsque vous créez un cluster AKS, la répartition des composants du plan de contrôle entre les zones de disponibilité n’est pas garantie. Vous pouvez ajouter des pools de nœuds supplémentaires à l’aide de la commande [az aks nodepool add][az-aks-nodepool-add] et spécifier `--zones` pour les nouveaux nœuds, mais cela ne changera pas la manière dont le plan de contrôle a été réparti entre les zones. Les paramètres de la zone de disponibilité peuvent être définis uniquement au moment de la création du cluster ou du pool de nœuds.

L’exemple suivant crée un cluster AKS nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup*. Un total de *3* nœuds sont créés : un agent dans la zone *1*, un dans la zone *2*, puis un dans la zone *3*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus2

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --node-count 3 \
    --zones 1 2 3
```

La création du cluster AKS ne prend que quelques minutes.

Lorsque vous décidez de la zone à laquelle un nouveau nœud doit appartenir, un pool de nœuds AKS donné utilise le [meilleur équilibrage des zones possible proposé par les groupes de machines virtuelles identiques sous-jacents][vmss-zone-balancing]. Un pool de nœuds AKS donné est considéré « équilibré » si le nombre de machines virtuelles de chaque zone est égal à celui de toutes les autres zones du groupe identique, à +\- 1 machine virtuelle près.

## <a name="verify-node-distribution-across-zones"></a>Vérifier la distribution des nœuds entre les zones

Lorsque le cluster est prêt, répertoriez les nœuds d'agent dans l’échelle définie pour voir dans quelle zone de disponibilité ils sont déployés.

Tout d’abord, obtenez les informations d’identification du cluster AKS à l’aide de la commande [az aks get-credentials][az-aks-get-credentials] :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Ensuite, utilisez la commande [kubectl describe][kubectl-describe] pour lister les nœuds du cluster et filtrer sur la valeur *failure-domain.beta.kubernetes.io/zone*. L’exemple suivant est destiné à un interpréteur de commandes Bash.

```console
kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"
```

L'exemple de sortie suivant montre les trois nœuds répartis dans la région et les zones de disponibilité spécifiées, notamment *eastus2-1* pour la première zone de disponibilité et *eastus2-2* pour la deuxième zone de disponibilité :

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
```

Lorsque vous ajoutez des nœuds supplémentaires à un pool d'agents, la plate-forme Azure distribue automatiquement les machines virtuelles sous-jacentes entre les zones de disponibilité spécifiées.

Notez que dans les versions plus récentes de Kubernetes (1.17.0 et ultérieures), AKS utilise l’étiquette plus récente `topology.kubernetes.io/zone` en plus de l’étiquette dépréciée `failure-domain.beta.kubernetes.io/zone`. Vous pouvez obtenir le même résultat que ci-dessus avec en exécutant le script suivant :

```console
kubectl get nodes -o custom-columns=NAME:'{.metadata.name}',REGION:'{.metadata.labels.topology\.kubernetes\.io/region}',ZONE:'{metadata.labels.topology\.kubernetes\.io/zone}'
```

Vous obtiendrez ainsi une sortie plus succincte :

```console
NAME                                REGION   ZONE
aks-nodepool1-34917322-vmss000000   eastus   eastus-1
aks-nodepool1-34917322-vmss000001   eastus   eastus-2
aks-nodepool1-34917322-vmss000002   eastus   eastus-3
```

## <a name="verify-pod-distribution-across-zones"></a>Vérifier la distribution des pods entre les zones

Comme décrit dans [Étiquettes, annotations et non-affinités connues][kubectl-well_known_labels], Kubernetes utilise l’étiquette `failure-domain.beta.kubernetes.io/zone` pour distribuer automatiquement les pods dans un contrôleur ou un service de réplication entre les différentes zones disponibles. Pour tester cela, vous pouvez effectuer un scale-up de votre cluster de 3 à 5 nœuds, pour vérifier que la propagation du pod est correcte :

```azurecli-interactive
az aks scale \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 5
```

Quand l’opération de mise à l’échelle se termine au bout de quelques minutes, la commande `kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"` doit produire une sortie similaire à cet exemple dans un interpréteur de commandes Bash :

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
Name:       aks-nodepool1-28993262-vmss000003
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000004
            failure-domain.beta.kubernetes.io/zone=eastus2-2
```

Nous avons maintenant deux nœuds supplémentaires dans les zones 1 et 2. Vous pouvez déployer une application composée de trois réplicas. Nous allons utiliser NGINX comme exemple :

```console
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

En affichant les nœuds où s’exécutent vos pods, vous voyez que les pods s’exécutent sur les nœuds correspondant aux trois différentes zones de disponibilité. Par exemple, avec la commande `kubectl describe pod | grep -e "^Name:" -e "^Node:"`, vous obtenez une sortie similaire à celle-ci dans un interpréteur de commandes Bash :

```console
Name:         nginx-6db489d4b7-ktdwg
Node:         aks-nodepool1-28993262-vmss000000/10.240.0.4
Name:         nginx-6db489d4b7-v7zvj
Node:         aks-nodepool1-28993262-vmss000002/10.240.0.6
Name:         nginx-6db489d4b7-xz6wj
Node:         aks-nodepool1-28993262-vmss000004/10.240.0.8
```

Comme vous pouvez le voir dans la sortie précédente, le premier pod s’exécute sur le nœud 0, qui se trouve dans la zone de disponibilité `eastus2-1`. Le deuxième pod s’exécute sur le nœud 2, qui correspond à `eastus2-3`, et le troisième dans le nœud 4, dans `eastus2-2`. Sans aucune configuration supplémentaire, Kubernetes répartit correctement les pods entre les trois zones de disponibilité.

## <a name="next-steps"></a>Étapes suivantes

Cet article explique comment créer un cluster AKS qui utilise des zones de disponibilité. Pour plus d’informations sur les clusters à haute disponibilité, consultez [Best practices for business continuity and disaster recovery in AKS][best-practices-bc-dr] (Meilleures pratiques pour la continuité d’activité et la récupération d’urgence dans AKS).

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-overview]: ../availability-zones/az-overview.md
[best-practices-bc-dr]: operator-best-practices-multi-region.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[standard-lb-limitations]: load-balancer-standard.md#limitations
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-nodepool-add]: /cli/azure/ext/aks-preview/aks/nodepool#ext-aks-preview-az-aks-nodepool-add
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[vmss-zone-balancing]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md#zone-balancing

<!-- LINKS - external -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-well_known_labels]: https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/
