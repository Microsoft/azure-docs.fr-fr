---
title: Utiliser des nœuds virtuels
titleSuffix: Azure Kubernetes Service
description: Vue d’ensemble de l’utilisation de nœuds virtuels avec Azure Kubernetes Services (AKS)
services: container-service
ms.topic: conceptual
ms.date: 09/21/2020
ms.custom: references_regions
ms.openlocfilehash: 2dd91e5c506f229d653fdf98bc0549c173cec793
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93351833"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes"></a>Créer et configurer un cluster Azure Kubernetes Service (AKS) pour utiliser des nœuds virtuels

Pour mettre à l’échelle rapidement des charges de travail d’application dans un cluster AKS, vous pouvez utiliser des nœuds virtuels. Les nœuds virtuels assurent un provisionnement rapide des pods et sont facturés à la seconde d’exécution. Il n’est pas nécessaire d’attendre que le gestionnaire de mise à l’échelle automatique du cluster Kubernetes déploie des nœuds de calcul de machine virtuelle pour exécuter les pods supplémentaires. Les nœuds virtuels sont uniquement pris en charge avec les nœuds et pods Linux.

Le module complémentaire Nœuds virtuels pour AKS est basé sur le projet open source [Virtual Kubelet][virtual-kubelet-repo].

Cet article vous donne une vue d’ensemble de la disponibilité par région et de la configuration réseau requise pour utiliser des nœuds virtuels ainsi que les limitations connues.

## <a name="regional-availability"></a>Disponibilité régionale

Toutes les régions dans lesquelles ACI prend en charge des références de réseau virtuel sont prises en charge pour les déploiements de nœuds virtuels.

Pour connaître les références de processeur et de mémoire disponibles dans chaque région, consultez l’article [Disponibilité des ressources Azure Container Instances pour Azure Container Instances dans les régions Azure - Groupes de conteneurs Linux](../container-instances/container-instances-region-availability.md#linux-container-groups)

## <a name="network-requirements"></a>Configuration requise pour le réseau

Les nœuds virtuels permettent la communication réseau entre les pods qui s’exécutent dans Azure Container Instances (ACI) et le cluster AKS. Pour que cette communication ait lieu, un sous-réseau de réseau virtuel est créé et des permissions déléguées sont assignées. Les nœuds virtuels ne fonctionnent qu’avec des clusters AKS créés à l’aide d’une mise en réseau *avancée* (Azure CNI). Par défaut, les clusters AKS sont créés avec une mise en réseau *de base* (kubenet).

Les pods exécutés dans Azure Container Instances (ACI) doivent accéder au point de terminaison de serveur d’API AKS afin de configurer la mise en réseau.

## <a name="known-limitations"></a>Limitations connues

Le fonctionnement des nœuds virtuel dépend fortement de l’ensemble de fonctionnalités d’ACI. Outre les quotas et les limites de [pour Azure Container Instances](../container-instances/container-instances-quotas.md), les scénarios suivants ne sont pas encore pris en charge avec les nœuds virtuels :

* Utilisation du principal du service pour extraire des images ACR. Une [solution de contournement](https://github.com/virtual-kubelet/azure-aci/blob/master/README.md#private-registry) consiste à utiliser les [Secrets Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)
* [Limitations du réseau virtuel](../container-instances/container-instances-vnet.md), dont le peering de réseaux virtuels, les stratégies réseau Kubernetes et le trafic sortant vers Internet avec les groupes de sécurité réseau.
* Initialiser les conteneurs
* [Alias d’hôte](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)
* [Arguments](../container-instances/container-instances-exec.md#restrictions) pour exécution dans ACI
* [DaemonSets](concepts-clusters-workloads.md#statefulsets-and-daemonsets) ne déploie pas les pods dans les nœuds virtuels.
* Les nœuds virtuels prennent en charge la planification des pods Linux. Vous pouvez installer manuellement le fournisseur d’[ACI Virtual Kubelet](https://github.com/virtual-kubelet/azure-aci) open source pour planifier des conteneurs Windows Server sur ACI.
* Les nœuds virtuels nécessitent des clusters AKS avec mise en réseau Azure CNI.
* Nœuds virtuels avec clusters privés.
* Utilisation des plages d’adresses IP autorisées du serveur d’API pour AKS.
* Le montage de volumes avec le partage Azure Files prend en charge les comptes [Usage général V1](../storage/common/storage-account-overview.md#types-of-storage-accounts). Suivez les instructions pour monter [un volume avec le partage Azure Files](azure-files-volume.md)

## <a name="next-steps"></a>Étapes suivantes

Configurez des nœuds virtuels pour vos clusters :

- [Créer des nœuds virtuels à l’aide d’Azure CLI](virtual-nodes-cli.md)
- [Créer des nœuds virtuels à l’aide du portail dans Azure Kubernetes Service (AKS)](virtual-nodes-portal.md)

Les nœuds virtuels constituent souvent l’un des composants d’une solution de mise à l’échelle dans AKS. Pour plus d’informations sur ces solutions, consultez les articles suivants :

- [Utiliser l’autoscaler de pods élastique Kubernetes][aks-hpa]
- [Utiliser l’autoscaler de cluster Kubernetes][aks-cluster-autoscaler]
- [Consultez l’exemple de mise à l’échelle automatique pour les nœuds virtuels][virtual-node-autoscale]
- [Découvrez-en plus sur la bibliothèque open source de Virtual Kubelet][virtual-kubelet-repo]

<!-- LINKS - external -->
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: ./cluster-autoscaler.md
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
