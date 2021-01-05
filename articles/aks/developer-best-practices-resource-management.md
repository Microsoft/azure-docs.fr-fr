---
title: Bonnes pratiques de gestion des ressources
titleSuffix: Azure Kubernetes Service
description: Découvrir les bonnes pratiques relatives à la gestion des ressources dans Azure Kubernetes Services (AKS) pour le développeur d’applications
services: container-service
author: zr-msft
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zarhoads
ms.openlocfilehash: 693cabac616dca8e108a2029c173a5e1b71c2695
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516733"
---
# <a name="best-practices-for-application-developers-to-manage-resources-in-azure-kubernetes-service-aks"></a>Bonnes pratiques relatives à la gestion des ressources dans Azure Kubernetes Services (AKS) pour le développeur d’applications

Quand vous développez et exécutez des applications dans Azure Kubernetes Service (AKS), vous devez tenir compte de quelques points clés. La façon dont vous gérez vos déploiements d’application peut impacter négativement l’expérience des utilisateurs finaux. Pour vous aider à réussir, gardez à l’esprit ces quelques bonnes pratiques relatives au développement et à l’exécution d’applications dans AKS.

Cet article traite des bonnes pratiques relatives à l’exécution d’un cluster et de charges de travail du point de vue d’un développeur d’applications. Pour accéder aux bonnes pratiques en matière d’administration, consultez les [bonnes pratiques de l’opérateur de cluster relatives à l’isolation et à la gestion de ressources dans Azure Kubernetes Service (AKS)][operator-best-practices-isolation]. Dans cet article, vous apprenez :

> [!div class="checklist"]
> * Ce que sont les demandes et limites de ressources de pod
> * Développer et déployer des applications avec Bridge to Kubernetes et Visual Studio Code
> * Comment utiliser l’outil `kube-advisor` pour rechercher les problèmes liés aux déploiements

## <a name="define-pod-resource-requests-and-limits"></a>À définir les demandes et limites de ressources de pod

**Guide sur les bonnes pratiques** : définissez des demandes et limites de pod sur tous les pods dans vos manifestes YAML. Si le cluster AKS utilise des *quotas de ressources*, votre déploiement peut être rejeté si vous ne définissez pas ces valeurs.

L’un des principaux moyens de gérer les ressources de calcul au sein d’un cluster AKS consiste à utiliser des demandes et des limites de pod. Ces demandes et limites indiquent au planificateur Kubernetes les ressources de calcul à affecter à un pod.

* Les **demandes d’UC/de mémoire pod** définissent une quantité fixe d’UC et de mémoire dont le pod a besoin régulièrement.
    * Quand le planificateur Kubernetes tente de placer un pod sur un nœud, les demandes de pod permettent de déterminer le nœud ayant suffisamment de ressources disponibles pour la planification.
    * Si vous ne définissez pas une requête de pod, elle est définie par défaut sur la limite définie.
    * Il est très important de surveiller les performances de votre application pour ajuster ces requêtes. En cas de demandes de ressource pod insuffisantes, votre application peut subir une dégradation des performances en raison de la surplanification d’un nœud. Si les requêtes sont surestimées, votre application peut avoir des difficultés à être planifiées.
* Les **limites de pod/de mémoire** représentent la quantité maximale d’UC et de mémoire qu’un pod peut utiliser. Les limites de mémoire permettent de définir les pods à supprimer en cas d’instabilité de nœud en raison de ressources insuffisantes. Sans les limites appropriées, les pods sont supprimés jusqu’à ce que la pression sur les ressources diminue. Un pod peut être ou non en mesure de dépasser la limite de processeur pendant un certain temps ; il ne sera pas supprimé s’il dépasse cette limite. 
    * Les limites de pod permettent de définir le moment où un pod perd le contrôle de la consommation des ressources. Lorsqu’une limite est dépassée, le pod est prioritaire pour la mise à jour de l’intégrité du nœud et réduire l’impact sur les pods partageant le nœud.
    * Si vous ne définissez pas de limite de pod, elle est définie par défaut sur la valeur disponible la plus élevée sur un nœud donné.
    * Ne définissez pas une limite de pod trop élevée ne pouvant pas être prise en charge par vos nœuds. Chaque nœud AKS réserve une quantité fixe d’UC et de mémoire pour les principaux composants de Kubernetes. Votre application peut tenter de consommer trop de ressources sur le nœud pour que les autres pods s’exécutent correctement.
    * Là encore, il est essentiel de superviser les performances de votre application à différents moments de la journée ou de la semaine. Déterminez les pics de demande et alignez les limites de pod avec les ressources requises pour répondre aux besoins maximum de l’application.

Dans les spécifications de votre pod, il est **recommandé et essentiel** de définir ces requêtes et limites en fonction des informations ci-dessus. Si vous n’incluez pas ces valeurs, le planificateur Kubernetes ne peut pas prendre en compte les ressources dont vos applications ont besoin pour faciliter la planification des décisions.

Si le planificateur place un pod sur un nœud dont les ressources sont insuffisantes, cela entraîne la dégradation des performances de l’application. Pour les administrateurs de cluster, il est vivement recommandé de définir des *quotas de ressources* sur un espace de noms qui vous oblige à définir des demandes et des limites de ressources. Pour plus d’informations, consultez [Quotas de ressources sur des clusters AKS][resource-quotas].

Quand vous définissez une demande ou une limite d’UC, la valeur est mesurée en unités d’UC. 
* Une UC de *1.0* équivaut à un cœur de processeur virtuel sous-jacent sur le nœud. 
* La même mesure est utilisée pour les GPU.
* Vous pouvez définir des fractions mesurées en millicœurs. Par exemple, *100 m* correspond à *0,1* pour un cœur de processeur virtuel sous-jacent.

Dans l’exemple de base suivant pour un pod NGINX, le pod demande *100m* de temps UC et *128Mi* de mémoire. Les limites de ressources pour le pod sont définies avec *250m* comme UC et *256Mi* comme mémoire :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

Pour plus d’informations sur les mesures et affectations de ressources, consultez [Gestion des ressources de calcul pour les conteneurs][k8s-resource-limits].

## <a name="develop-and-debug-applications-against-an-aks-cluster"></a>Développer et déboguer des applications sur un cluster AKS

**Guide de bonne pratique** : les équipes de développement doivent déployer et déboguer leurs applications sur un cluster AKS à l’aide de Bridge to Kubernetes.

Avec Bridge to Kubernetes, vous pouvez développer, déboguer et tester les applications directement sur un cluster AKS. Les développeurs d’une équipe travaillent ensemble pour générer et tester l’application tout au long de son cycle de vie. Vous pouvez continuer à utiliser des outils existants tels que Visual Studio ou Visual Studio Code. Une extension est installée pour Bridge to Kubernetes et vous permet de développer directement dans un cluster AKS.

Ce processus de développement et de test intégré à Bridge to Kubernetes évite de recourir à des environnements de test locaux tels que [minikube][minikube]. Au lieu de cela, vous développez et testez votre application sur un cluster AKS. Ce cluster peut être sécurisé et isolé, comme indiqué dans la section précédente sur l’utilisation d’espaces de noms pour isoler logiquement un cluster.

Bridge to Kubernetes est destiné à être utilisé avec les applications qui s’exécutent sur les nœuds et pods Linux.

## <a name="use-the-visual-studio-code-extension-for-kubernetes"></a>Utiliser l’extension Visual Studio Code pour Kubernetes

**Guide sur les bonnes pratiques** : installez et utilisez l’extension VS Code pour Kubernetes quand vous écrivez des manifestes YAML. Vous pouvez également utiliser l’extension en tant que solution de déploiement intégrée, ce qui peut être utile pour les propriétaires d’applications qui interagissent rarement avec le cluster AKS.

L’[extension Visual Studio Code pour Kubernetes][vscode-kubernetes] vous permet de développer et déployer des applications sur AKS. L’extension fournit les informations IntelliSense pour les ressources Kubernetes ainsi que les graphiques et modèles Helm. Vous pouvez également parcourir, déployer et modifier des ressources Kubernetes à partir de VS Code. L’extension fournit une vérification IntelliSense pour les demandes ou limites de ressources définies dans les spécifications de pod :

![Avertissement de l’extension VS Code pour Kubernetes concernant des limites de mémoire manquantes](media/developer-best-practices-resource-management/vs-code-kubernetes-extension.png)

## <a name="regularly-check-for-application-issues-with-kube-advisor"></a>Rechercher régulièrement les problèmes liés à l’application avec kube-advisor

**Meilleure pratique** : exécutez régulièrement la dernière version de l’outil open source `kube-advisor` pour détecter les problèmes dans votre cluster. Si vous appliquez des quotas de ressources sur un cluster AKS existant, exécutez d’abord `kube-advisor` pour rechercher les pods dont les demandes et limites de ressources ne sont pas définies.

L’outil [kube-advisor][kube-advisor] est un projet open source AKS associé qui analyse un cluster Kubernetes et signale les problèmes trouvés. Une vérification utile consiste à identifier les pods dépourvus de demandes et de limites de ressources.

L’outil kube-advisor peut rendre compte des limites et demandes de ressources manquantes dans PodSpecs pour les applications Windows et Linux, mais l’outil kube-advisor proprement dit doit être planifié sur un pod Linux. Vous pouvez planifier un pod pour qu’il s’exécute sur un pool de nœuds avec un système d’exploitation spécifique à l’aide d’un [sélecteur de nœud][k8s-node-selector] dans la configuration du pod.

Dans un cluster AKS qui héberge plusieurs applications et équipes de développement, il peut être difficile de suivre les pods sans ces demandes et limites de ressources. Une bonne pratique consiste à exécuter régulièrement `kube-advisor` sur vos clusters AKS.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons traité des bonnes pratiques relatives à l’exécution d’un cluster et de charges de travail du point de vue d’un opérateur de cluster. Pour accéder aux bonnes pratiques en matière d’administration, consultez les [bonnes pratiques de l’opérateur de cluster relatives à l’isolation et à la gestion de ressources dans Azure Kubernetes Service (AKS)][operator-best-practices-isolation].

Pour implémenter quelques-unes de ces bonnes pratiques, consultez les articles suivants :

* [Développer avec Bridge to Kubernetes][btk]
* [Rechercher des problèmes avec kube-advisor][aks-kubeadvisor]

<!-- EXTERNAL LINKS -->
[k8s-resource-limits]: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
[vscode-kubernetes]: https://github.com/Azure/vscode-kubernetes-tools
[kube-advisor]: https://github.com/Azure/kube-advisor
[minikube]: https://kubernetes.io/docs/setup/minikube/

<!-- INTERNAL LINKS -->
[aks-kubeadvisor]: kube-advisor-tool.md
[btk]: /visualstudio/containers/overview-bridge-to-kubernetes
[operator-best-practices-isolation]: operator-best-practices-cluster-isolation.md
[resource-quotas]: operator-best-practices-scheduler.md#enforce-resource-quotas
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
