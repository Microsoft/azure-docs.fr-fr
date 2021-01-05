---
title: Concepts - Mettre à l’échelle des applications dans Azure Kubernetes Service (AKS)
description: Découvrez la mise à l’échelle dans Azure Kubernetes Service (AKS), notamment l’autoscaler de pods élastique, l’autoscaler de cluster et le connecteur Azure Container Instances.
services: container-service
ms.topic: conceptual
ms.date: 02/28/2019
ms.openlocfilehash: b72ed7cefc6a16eb484e1337dbd64e5f069a2201
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686036"
---
# <a name="scaling-options-for-applications-in-azure-kubernetes-service-aks"></a>Options de mise à l’échelle des applications dans AKS (Azure Kubernetes Service)

Lorsque vous exécutez des applications dans Azure Kubernetes Service (AKS), vous pouvez avoir besoin d’augmenter ou de diminuer la quantité de ressources de calcul. Tout comme le nombre des instances d’application qui vous est nécessaire change, le nombre de nœuds Kubernetes sous-jacents peut également varier. Par ailleurs, vous pouvez avoir besoin d’approvisionner rapidement un grand nombre d’instances d’application supplémentaires.

Cet article présente les concepts fondamentaux qui vous aident à mettre à l’échelle vos applications dans AKS :

- [Mise à l’échelle manuelle](#manually-scale-pods-or-nodes)
- [Autoscaler de pods élastique](#horizontal-pod-autoscaler)
- [Autoscaler de cluster](#cluster-autoscaler)
- [Intégration d’Azure Container Instances (ACI) à AKS](#burst-to-azure-container-instances)

## <a name="manually-scale-pods-or-nodes"></a>Mettre à l’échelle des pods ou des nœuds manuellement

Vous pouvez mettre à l’échelle des réplicas (pods) et des nœuds manuellement pour tester la façon dont votre application répond à une modification au niveau des ressources disponibles et de l’état. Adapter manuellement les ressources vous permet également de définir une quantité donnée de ressources à utiliser pour maintenir un coût fixe, par exemple le nombre de nœuds. Pour mettre à l’échelle manuellement, vous définissez le nombre de réplicas ou de nœuds. L’API Kubernetes planifie ensuite la création de pods supplémentaires ou le drainage de nœuds en fonction du nombre de réplicas ou de nœuds.

Lors d’un scale-down des nœuds, l’API Kubernetes appelle l’API de calcul Azure appropriée liée au type de calcul utilisé par votre cluster. Par exemple, pour les clusters basés sur VM Scale Sets, la logique de sélection des nœuds à supprimer est déterminée par l’API VM Scale Sets. Pour en savoir plus sur la façon dont les nœuds sont sélectionnés pour la suppression lors d’un scale-down, consultez les [Questions fréquentes (FAQ) sur VMSS](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#if-i-reduce-my-scale-set-capacity-from-20-to-15-which-vms-are-removed).

Pour vous familiariser avec la mise à l’échelle manuelle des pods et des nœuds, consultez [Mettre à l’échelle des applications dans AKS][aks-scale].

## <a name="horizontal-pod-autoscaler"></a>Autoscaler de pods élastique

Kubernetes utilise l’autoscaler de pods élastique pour superviser la demande en ressources et adapter automatiquement le nombre de réplicas. Par défaut, l’autoscaler de pods élastique vérifie l’API de métriques toutes les 30 secondes à la recherche d’une modification à apporter au nombre de réplicas. Lorsque des modifications sont nécessaires, le nombre de réplicas est augmenté ou diminué en conséquence. L’autoscaler de pods élastique fonctionne avec les clusters AKS qui ont déployé le serveur de métriques pour Kubernetes 1.8 +.

![Mise à l’échelle de pods élastique Kubernetes](media/concepts-scale/horizontal-pod-autoscaling.png)

Lorsque vous configurez l’autoscaler de pods élastique pour un déploiement donné, vous définissez le nombre minimal et maximal de réplicas qui peuvent s’exécuter. Vous précisez également la métrique à surveiller et sur laquelle baser vos décisions de mise à l’échelle, par exemple l’utilisation du processeur.

Pour vous familiariser avec l’autoscaler de pods élastique dans AKS, consultez [Mettre à l’échelle des pods dans AKS][aks-hpa].

### <a name="cooldown-of-scaling-events"></a>Ralentissement des événements de mise à l’échelle

Étant donné que l’autoscaler de pods élastique vérifie l’API de métriques toutes les 30 secondes, les événements de mise à l’échelle précédents peuvent ne pas être totalement terminés avant la vérification suivante. Ce comportement peut pousser l’autoscaler de pods élastique à modifier le nombre de réplicas avant même que l’événement de mise à l’échelle précédent ait pu recevoir la charge de travail de l’application, et que les demandes en ressources soient ajustées en conséquence.

Pour réduire le nombre d’événements de concurrence, une valeur de délai est définie. Cette valeur précise la durée pendant laquelle l’autoscaler de pods élastique doit attendre, entre la fin d’un événement de mise à l’échelle et le déclenchement d’un autre événement de mise à l’échelle. Ce comportement permet au nouveau nombre de réplicas d’être pris en compte, et à l’API de métriques de refléter la charge de travail distribuée. Il n’y a [aucun délai pour les événements de scale-up à partir de Kubernetes 1.12](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-cooldown-delay). Toutefois, le délai des événements de scale-down est de 5 minutes par défaut.

Actuellement, vous ne pouvez pas paramétrer ces valeurs de ralentissement à partir de la valeur par défaut.

## <a name="cluster-autoscaler"></a>Autoscaler de cluster

Pour répondre aux demandes changeantes de pods, Kubernetes propose un autoscaler de cluster qui ajuste le nombre de nœuds en fonction des ressources de calcul demandées dans le pool de nœuds. Par défaut, l’autoscaler de cluster vérifie le serveur d’API de métriques toutes les 10 secondes à la recherche de toute modification à apporter au nombre de nœuds. Si l’autoscaler de cluster détermine qu’une modification est nécessaire, le nombre de nœuds de votre cluster AKS est augmenté ou diminué en conséquence. L’autoscaler de cluster fonctionne avec les clusters AKS avec RBAC Kubernetes qui exécutent Kubernetes versions 1.10.x ou ultérieures.

![Autoscaler de cluster Kubernetes](media/concepts-scale/cluster-autoscaler.png)

L’autoscaler de cluster est généralement utilisé parallèlement à l’autoscaler de pods élastique. Lorsqu’ils sont combinés, l’autoscaler de pods élastique augmente ou diminue le nombre de pods en fonction de l’exigence des applications, tandis que l’autoscaler de cluster ajuste à proportion le nombre de nœuds nécessaires pour exécuter ces pods supplémentaires.

Pour vous familiariser avec l’autoscaler de cluster dans AKS, consultez [Autoscaler de cluster sur AKS][aks-cluster-autoscaler].

### <a name="scale-out-events"></a>Événements de scale-out

Si les ressources de calcul d’un nœud sont insuffisantes pour l’exécution d’un pod demandé, ce pod ne peut pas avancer dans le processus de planification. Le pod ne peut pas démarrer, sauf si des ressources de calcul supplémentaires sont disponibles dans le pool de nœuds.

Lorsque l’autoscaler de cluster remarque que des pods ne sont pas panifiables en raison de contraintes liées aux ressources du pool de nœuds, le nombre de nœuds à l’intérieur de ce pool est augmenté pour fournir les ressources de calcul supplémentaires. Lorsque ces nœuds supplémentaires sont correctement déployés et utilisables au sein du pool de nœuds, les pods sont alors planifiés pour s’exécuter sur eux.

Si votre application doit rapidement se mettre à l'échelle, il est possible que certains pods restent en état d’attente de planification, jusqu’à ce que les nœuds supplémentaires déployés par l’autoscaler de cluster puissent accepter les pods planifiés. Pour les applications qui présentent des demandes de croissance extrêmement forte et rapide, vous pouvez mettre à l’échelle au moyen de nœuds virtuels et d’Azure Container Instances.

### <a name="scale-in-events"></a>Événements de scale-in

L’autoscaler de cluster surveille également le statut de planification des pods pour les nœuds qui n’ont pas reçu récemment de nouvelles demandes de planification. Ce scénario indique que le pool de nœuds détient plus de ressources de calcul que nécessaire, et que le nombre de nœuds peut être réduit.

Un nœud, qui transmet un seuil indiquant pendant 10 minutes (par défaut) qu’il n’est plus nécessaire, est planifié pour la suppression. Lorsque cette situation se produit, les pods sont planifiés pour s’exécuter sur d’autres nœuds au sein du pool de nœuds tandis que l’autoscaler de cluster réduit le nombre de nœuds.

Vos applications peuvent rencontrer quelques perturbations au moment où les pods sont planifiés sur des nœuds différents et que l’autoscaler de cluster diminue le nombre de nœuds. Pour limiter ces perturbations, évitez les applications qui utilisent une seule instance de pod.

## <a name="burst-to-azure-container-instances"></a>Intégration à Azure Container Instances

Pour faire évoluer rapidement votre cluster AKS, vous pouvez intégrer Azure Container Instances (ACI). Kubernetes dispose de composants intégrés pour mettre à l’échelle le nombre de réplicas et de nœuds. Toutefois, si votre application doit rapidement s’adapter, l’autoscaler de pods élastique peut planifier plus de pods que la quantité pouvant être fournie par les ressources de calcul existantes dans le pool de nœuds. S’il est configuré, ce scénario oblige par déclenchement l’autoscaler de cluster à déployer des nœuds supplémentaires dans le pool de nœuds, mais quelques minutes peuvent être nécessaires pour que ces nœuds soient correctement provisionnés et permettent au planificateur Kubernetes d’exécuter des pods dessus.

![Mise à l'échelle rapide de Kubernetes sur ACI](media/concepts-scale/burst-scaling.png)

ACI vous permet de déployer rapidement des instances de conteneur sans la surcharge d’une infrastructure supplémentaire. Lorsque vous vous connectez à AKS, ACI devient une extension logique et sécurisée de votre cluster AKS. Le composant [nœuds virtuels][virtual-nodes-cli], qui est basé sur [Virtual Kubelet][virtual-kubelet], est installé dans votre cluster AKS qui présente ACI comme un nœud Kubernetes virtuel. Kubernetes peut alors planifier les pods s’exécutant en tant qu’instances ACI via des nœuds virtuels, et non en tant que pods sur des nœuds de machine virtuelle, directement dans votre cluster AKS.

Votre application n’a besoin d’aucune modification pour utiliser les nœuds virtuels. Les déploiements peuvent mettre à l’échelle dans AKS et ACI, et sans aucun délai car l’autoscaler de cluster déploie les nouveaux nœuds dans votre cluster AKS.

Les nœuds virtuels sont déployés sur un sous-réseau supplémentaire, dans le même réseau virtuel que votre cluster AKS. Cette configuration de réseau virtuel permet au trafic entre ACI et AKS d’être sécurisé. À l’instar d’un cluster AKS, une instance ACI est une ressource de calcul logique, sécurisée, qui est isolée des autres utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

Pour vous familiariser avec la mise à l’échelle des applications, suivez tout d’abord le [Guide de démarrage rapide pour créer un cluster AKS avec l’interface CLI][aks-quickstart]. Vous pouvez ensuite commencer à mettre à l’échelle les applications manuellement ou automatiquement dans votre cluster AKS :

- Mettre à l’échelle des [pods][aks-manually-scale-pods] ou des [nœuds][aks-manually-scale-nodes] manuellement
- Utiliser l’[autoscaler de pods élastique][aks-hpa]
- Utiliser l’[autoscaler de cluster][aks-cluster-autoscaler]

Pour plus d’informations sur les concepts fondamentaux de Kubernetes et d’AKS, consultez les articles suivants :

- [Clusters et charges de travail Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Accès et identité Kubernetes/AKS][aks-concepts-identity]
- [Sécurité Kubernetes/AKS][aks-concepts-security]
- [Réseaux virtuels Kubernetes/AKS][aks-concepts-network]
- [Stockage Kubernetes/AKS][aks-concepts-storage]

<!-- LINKS - external -->
[virtual-kubelet]: https://virtual-kubelet.io/

<!-- LINKS - internal -->
[aks-quickstart]: kubernetes-walkthrough.md
[aks-hpa]: tutorial-kubernetes-scale.md#autoscale-pods
[aks-scale]: tutorial-kubernetes-scale.md
[aks-manually-scale-pods]: tutorial-kubernetes-scale.md#manually-scale-pods
[aks-manually-scale-nodes]: tutorial-kubernetes-scale.md#manually-scale-aks-nodes
[aks-cluster-autoscaler]: ./cluster-autoscaler.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-network]: concepts-network.md
[virtual-nodes-cli]: virtual-nodes-cli.md
