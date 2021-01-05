---
title: Bonnes pratiques relatives aux fonctionnalités du planificateur
titleSuffix: Azure Kubernetes Service
description: Découvrir les bonnes pratiques de l’opérateur relatives à l’utilisation des fonctionnalités avancées du planificateur, notamment les teintes (taints) et tolérances (tolerations), les sélecteurs et l’affinité de nœud, ainsi que l’affinité ou l’anti-affinité entre pods dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: 1a8138b4b2fdab2cdef8d2cb4c27de8d12ef38cd
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107344"
---
# <a name="best-practices-for-advanced-scheduler-features-in-azure-kubernetes-service-aks"></a>Bonnes pratiques relatives aux fonctionnalités avancées du planificateur dans Azure Kubernetes Service (AKS)

Quand vous gérez des clusters dans Azure Kubernetes Service (AKS), il est souvent nécessaire d’isoler les équipes et les charges de travail. Le planificateur Kubernetes offre des fonctionnalités avancées avec lesquelles vous pouvez contrôler les pods susceptibles d’être planifiés sur certains nœuds et la distribution adéquate des applications multipods dans le cluster. 

Cet article traite des bonnes pratiques relatives aux fonctionnalités de planification Kubernetes avancées pour les opérateurs de clusters. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Utiliser des teintes (taints) et des tolérances (tolerations) pour limiter les pods pouvant être planifiés sur des nœuds
> * Privilégier l’exécution de pods sur certains nœuds avec des sélecteurs de nœud ou l’affinité de nœud
> * Diviser ou regrouper des pods avec l’affinité ou l’anti-affinité entre pods

## <a name="provide-dedicated-nodes-using-taints-and-tolerations"></a>Fournir des nœuds dédiés à l’aide de teintes et de tolérances

**Guide de bonne pratique** : limitez l’accès des applications nécessitant de nombreuses ressources, comme les contrôleurs d’entrée, à des nœuds spécifiques. Conservez les ressources de nœud disponibles pour les charges de travail qui en ont besoin et n’autorisez pas la planification d’autres charges de travail sur les nœuds.

Quand vous créez votre cluster AKS, vous pouvez déployer des nœuds avec prise en charge des unités centrales graphiques (GPU) ou un grand nombre de processeurs puissants. Ces nœuds sont souvent utilisés pour les charges de travail de traitement de données volumineuses, notamment celles liées au machine learning (ML) ou à l’intelligence artificielle (IA). Ce type de matériel étant généralement une ressource de nœud coûteuse à déployer, limitez les charges de travail qui peuvent être planifiées sur ces nœuds. Au lieu de cela, vous pouvez dédier certains nœuds du cluster à l’exécution de services d’entrée et empêcher les autres charges de travail.

Cette prise en charge pour différents nœuds est fournie à l’aide de plusieurs pools de nœuds. Un cluster AKS fournit un ou plusieurs pools de nœud.

Le planificateur Kubernetes peut utiliser des teintes et des tolérances pour restreindre les charges de travail qui peuvent s’exécuter sur des nœuds.

* Quand une **teinte** est appliquée à un nœud, seuls des pods spécifiques peuvent être planifiés sur le nœud.
* Une **tolérance** est ensuite appliquée à un pod pour lui permettre de *tolérer* la teinte d’un nœud.

Quand vous déployez un pod sur un cluster AKS, Kubernetes planifie uniquement des pods sur les nœuds où une tolérance est alignée avec la teinte. Par exemple, vous avez un pool de nœuds dans votre cluster AKS pour les nœuds avec prise en charge des GPU. Vous définissez un nom, comme *gpu*, puis une valeur pour la planification. Si vous affectez à cette valeur *NoSchedule*, le planificateur Kubernetes ne peut pas planifier de pods sur le nœud si le pod ne définit pas la tolérance appropriée.

```console
kubectl taint node aks-nodepool1 sku=gpu:NoSchedule
```

Après avoir appliqué une teinte aux nœuds, vous définissez une tolérance dans la spécification de pod qui autorise la planification sur les nœuds. L’exemple suivant définit `sku: gpu` et `effect: NoSchedule` pour tolérer la teinte appliquée au nœud à l’étape précédente :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Quand ce pod est déployé, par exemple en utilisant `kubectl apply -f gpu-toleration.yaml`, Kubernetes peut planifier le pod sur les nœuds avec la teinte appliquée. Cette isolation logique vous permet de contrôler l’accès aux ressources au sein d’un cluster.

Quand vous appliquez des teintes, collaborez avec les développeurs et propriétaires d’applications pour leur permettre de définir les tolérances requises dans leurs déploiements.

Pour en savoir plus sur l’utilisation de plusieurs pools de nœuds dans AKS, voir [Créer et gérer plusieurs pools de nœuds pour un cluster dans AKS][use-multiple-node-pools].

### <a name="behavior-of-taints-and-tolerations-in-aks"></a>Comportement des teintes et des tolérances dans AKS

Lorsque vous mettez à niveau un pool de nœuds dans AKS, les teintes et les tolérances suivent un modèle défini lorsqu’elles sont appliquées à de nouveaux nœuds :

- **Clusters par défaut qui utilisent des groupes de machines virtuelles identiques**
  - Vous pouvez [appliquer une teinte à un pool de nœuds][taint-node-pool] à partir de l’API AKS afin que les nœuds qui viennent d’être mis à l’échelle reçoivent des teintes de nœud spécifiés par l’API.
  - Partons du principe que nous disposons d’un cluster à deux nœuds : *node1* et *node2*. Vous mettez à niveau le pool de nœuds.
  - Deux nœuds supplémentaires sont créés, *node3* et *node4*, et les teintes y sont envoyées.
  - Les *node1* et *node2* originaux sont supprimés.

- **Clusters par défaut sans prise en charge de groupe de machine virtuelle identique**
  - Encore une fois, partons du principe que nous disposons d’un cluster à deux nœuds : *node1* et *node2*. Lorsque vous le mettez à niveau, un nœud supplémentaire (*node3*) est créé.
  - Les teintes du *node1* sont appliquées au *node3*, avant que *node1* soit supprimé.
  - Un nouveau nœud est créé (nommé *node1*, car le précédent *node1* a été supprimé), et les teintes du *node2* sont appliquées au nouveau *node1*. Ensuite, *node2* est supprimé.
  - Fondamentalement, *node1* devient *node3* et *node2* devient *node1*.

Lorsque vous faites évoluer un pool de nœuds dans AKS, les teintes et les tolérances ne sont pas transmises par défaut.

## <a name="control-pod-scheduling-using-node-selectors-and-affinity"></a>Contrôler la planification des pods à l’aide de sélecteurs de nœud et de l’affinité de nœud

**Guide de bonne pratique** : contrôlez la planification des pods sur les nœuds à l’aide de sélecteurs de nœud, de l’affinité de nœud ou de l’affinité entre pods. Ces paramètres permettent au planificateur Kubernetes d’isoler logiquement les charges de travail, notamment par matériel dans le nœud.

Les teintes et tolérances servent à isoler logiquement les ressources avec une coupure dure : si le pod ne tolère pas la teinte d’un nœud, il n’est pas planifié sur le nœud. Une autre approche consiste à utiliser des sélecteurs de nœud. Vous étiquetez les nœuds, par exemple pour indiquer un stockage SSD attaché localement ou une grande quantité de mémoire, puis définissez un sélecteur de nœud dans la spécification de pod. Kubernetes planifie ensuite ces pods sur un nœud correspondant. Contrairement aux tolérances, les pods sans sélecteur de nœud correspondant peuvent être planifiés sur des nœuds étiquetés. Ce comportement autorise la consommation des ressources inutilisées sur les nœuds, mais donne la priorité aux pods qui définissent le sélecteur de nœud correspondant.

Examinons un exemple de nœuds avec une grande quantité de mémoire. Ces nœuds peuvent privilégier les pods qui demandent une grande quantité de mémoire. Pour que les ressources ne restent pas inactives, d’autres pods sont également autorisés à s’exécuter.

```console
kubectl label node aks-nodepool1 hardware=highmem
```

Une spécification de pod ajoute ensuite la propriété `nodeSelector` pour définir un sélecteur de nœud qui correspond à l’étiquette définie sur un nœud :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  nodeSelector:
      hardware: highmem
```

Quand vous utilisez ces options du planificateur, collaborez avec les propriétaires et développeurs d’applications pour leur permettre de définir correctement leurs spécifications de pod.

Pour plus d’informations sur l’utilisation de sélecteurs de nœud, consultez [Affectation de pods à des nœuds][k8s-node-selector].

### <a name="node-affinity"></a>Affinité de nœud

Un sélecteur de nœud est un moyen simple d’affecter des pods à un nœud donné. Pour plus de flexibilité, utilisez l’*affinité de nœud*. Avec l’affinité de nœud, vous définissez ce qui se passe si le pod ne peut pas être mis en correspondance avec un nœud. Vous pouvez *exiger* que le planificateur Kubernetes mette en correspondance un pod avec un hôte étiqueté. Vous pouvez également *préférer* une correspondance, mais autoriser la planification du pod sur un autre hôte si aucune correspondance n’est disponible.

L’exemple suivant définit l’affinité de nœud avec la valeur *requiredDuringSchedulingIgnoredDuringExecution*. Selon cette affinité, la planification Kubernetes doit utiliser un nœud avec une étiquette correspondante. Si aucun nœud n’est disponible, le pod doit attendre pour que planification puisse se poursuivre. Pour autoriser la planification du pod sur un autre nœud, vous pouvez définir la valeur *preferredDuringSchedulingIgnoreDuringExecution* :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values: highmem
```

La partie *IgnoredDuringExecution* du paramètre indique que le pod ne doit pas être supprimé du nœud si ses étiquettes viennent à changer. Le planificateur Kubernetes n’utilise les étiquettes de nœud mises à jour que pour les nouveaux pods en cours de planification, pas pour ceux déjà planifiés sur les nœuds.

Pour plus d’informations, consultez [Affinité et anti-affinité][k8s-affinity].

### <a name="inter-pod-affinity-and-anti-affinity"></a>Affinité et anti-affinité entre pods

Une dernière approche, employée par le planificateur Kubernetes pour isoler logiquement des charges de travail, consiste à utiliser l’affinité ou l’anti-affinité entre pods. Les paramètres définissent si *oui* ou *non* des pods peuvent être planifiés sur un nœud ayant un pod correspondant. Par défaut, le planificateur Kubernetes essaie de planifier plusieurs pods dans un jeu de réplicas sur les nœuds. Vous pouvez définir des règles plus spécifiques autour de ce comportement.

Un bon exemple est une application web qui utilise également un Cache Azure pour Redis. Vous pouvez utiliser des règles d’anti-affinité de pod pour demander au planificateur Kubernetes de distribuer les réplicas sur les nœuds. Vous pouvez ensuite utiliser des règles d’affinité pour que chaque composant de l’application web soit planifié sur le même hôte qu’un cache correspondant. La distribution des pods sur les nœuds est similaire à celle de l’exemple suivant :

| **Nœud 1** | **Nœud 2** | **Nœud 3** |
|------------|------------|------------|
| webapp-1   | webapp-2   | webapp-3   |
| cache-1    | cache-2    | cache-3    |

Cet exemple de déploiement plus complexe dépasse le cadre de l’utilisation des sélecteurs ou de l’affinité de nœud. Le déploiement vous permet de contrôler la façon dont Kubernetes planifie les pods sur les nœuds et dont il isole logiquement les ressources. Pour obtenir un exemple complet de cette application web avec un exemple Azure Cache pour Redis, consultez [Colocaliser des pods sur le même nœud][k8s-pod-affinity].

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons traité des fonctionnalités avancées du planificateur Kubernetes. Pour plus d’informations sur les opérations liées aux clusters dans AKS, consultez les bonnes pratiques suivantes :

* [Multilocation et isolation de cluster][aks-best-practices-scheduler]
* [Fonctionnalités de base du planificateur Kubernetes][aks-best-practices-scheduler]
* [Authentification et autorisation][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-taints-tolerations]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[k8s-node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[k8s-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
[k8s-pod-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#always-co-located-in-the-same-node

<!-- INTERNAL LINKS -->
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[taint-node-pool]: use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool
