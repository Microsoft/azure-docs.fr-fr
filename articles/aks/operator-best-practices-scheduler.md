---
title: Bonnes pratiques de l’opérateur - Fonctionnalités de base du planificateur dans Azure Kubernetes Service (AKS)
description: Découvrir les bonnes pratiques de l’opérateur relatives aux fonctionnalités de base du planificateur, notamment les quotas de ressources et les budgets de perturbation de pod dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: 087c1d2efc93b8460a3683a4e66916d73fd4e885
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87015678"
---
# <a name="best-practices-for-basic-scheduler-features-in-azure-kubernetes-service-aks"></a>Bonnes pratiques relatives aux fonctionnalités de base du planificateur dans Azure Kubernetes Service (AKS)

Quand vous gérez des clusters dans Azure Kubernetes Service (AKS), il est souvent nécessaire d’isoler les équipes et les charges de travail. Le planificateur Kubernetes offre des fonctionnalités qui vous permettent de contrôler la distribution des ressources de calcul ou de limiter l’impact des événements de maintenance.

Cet article traite des bonnes pratiques relatives aux fonctionnalités de planification Kubernetes de base pour les opérateurs de clusters. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Utiliser des quotas d’utilisation de ressources pour fournir une quantité fixe de ressources à des équipes ou charges de travail
> * Limiter l’impact de la maintenance planifiée à l’aide de budgets de perturbation de pod
> * Rechercher les pods dépourvus de demandes et de limites de ressources à l’aide de l’outil `kube-advisor`

## <a name="enforce-resource-quotas"></a>Appliquer des quotas de ressources

**Guide de bonne pratique** : planifiez et appliquez des quotas de ressources au niveau de l’espace de noms. Si les pods ne définissent pas de demandes ni de limites de ressources, rejetez le déploiement. Supervisez l’utilisation des ressources et ajustez les quotas en fonction des besoins.

Les demandes et limites de ressources sont placées dans la spécification de pod. Ces limites sont utilisées par le planificateur Kubernetes au moment du déploiement pour rechercher un nœud disponible dans le cluster. Ces demandes et limites s’appliquent au niveau de chaque pod. Pour plus d’informations sur la façon de définir ces valeurs, consultez [Définir des demandes et limites de pod][resource-limits].

Pour fournir un moyen de réserver et de limiter les ressources à l’échelle d’un projet ou d’une équipe de développement, vous devez utiliser des *quotas de ressources*. Ces quotas sont définis sur un espace de noms et peuvent être définis en fonction des éléments suivants :

* **Ressources de calcul** (par exemple, le processeur et la mémoire ou des GPU).
* **Ressources de stockage** (inclut le nombre total de volumes ou la quantité d’espace disque pour une classe de stockage donnée).
* **Nombre d’objets** (par exemple, le nombre maximal de secrets, de services ou de travaux pouvant être créés).

Kubernetes ne sollicite pas excessivement les ressources. Si le total cumulé des demandes ou des limites de ressources dépasse le quota affecté, tout nouveau déploiement échoue.

Quand vous définissez des quotas de ressources, tous les pods créés dans l’espace de noms doivent fournir des demandes ou des limites dans leurs spécifications de pod. S’ils ne fournissent pas ces valeurs, vous pouvez refuser le déploiement. Au lieu de cela, vous pouvez [configurer des demandes et limites par défaut pour un espace de noms][configure-default-quotas].

L’exemple de manifeste YAML suivant nommé *dev-app-team-quotas.yaml* définit une limite inconditionnelle avec, au total, *10* processeurs, *20Gi* de mémoire et *10* pods :

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-app-team
spec:
  hard:
    cpu: "10"
    memory: 20Gi
    pods: "10"
```

Ce quota de ressources peut être appliqué en spécifiant l’espace de noms, par exemple *dev-apps* :

```console
kubectl apply -f dev-app-team-quotas.yaml --namespace dev-apps
```

Collaborez avec les propriétaires et développeurs d’application pour comprendre leurs besoins et appliquer les quotas de ressources appropriés.

Pour plus d’informations sur les objets de ressource, étendues et priorités disponibles, consultez [Quotas de ressources dans Kubernetes][k8s-resource-quotas].

## <a name="plan-for-availability-using-pod-disruption-budgets"></a>Planifier la disponibilité à l’aide de budgets de perturbation de pod

**Guide de bonne pratique** : pour assurer la disponibilité des applications, définissez des budgets de perturbation de pod (PDB). Ceux-ci garantissent un nombre minimal de pods disponibles dans le cluster.

Deux événements perturbateurs peuvent entraîner la suppression de pods :

* *Perturbations involontaires* : il s’agit d’événements qui échappent généralement au contrôle de l’opérateur de cluster ou du propriétaire d’application.
  * Citons en exemple une défaillance matérielle sur la machine physique, une alerte sur le noyau ou la suppression d’une machine virtuelle de nœud.
* *Perturbations volontaires* : il s’agit d’événements demandés par l’opérateur de cluster ou le propriétaire d’application.
  * Citons en exemple la mise à niveau d’un cluster, la mise à jour d’un modèle de déploiement ou la suppression accidentelle d’un pod.

Pour atténuer les perturbations involontaires, vous pouvez utiliser plusieurs réplicas de vos pods dans un déploiement. L’exécution de plusieurs nœuds dans le cluster AKS peut aussi contribuer à réduire les perturbations involontaires. Pour les perturbations volontaires, Kubernetes fournit des *budgets de perturbation de pod*. Ceux-ci permettent à l’opérateur de cluster de définir le nombre minimal de ressources disponibles ou le nombre maximal de ressources indisponibles. Les budgets de perturbation de pod vous permettent de planifier la réponse des déploiements ou des jeux de réplicas à la suite d’un événement perturbateur.

Si un cluster doit être mis à niveau ou qu’un modèle de déploiement doit être mis à jour, le planificateur Kubernetes fait en sorte que des pods supplémentaires sont planifiés sur d’autres nœuds avant que les événements perturbateurs volontaires ne se poursuivent. Avant le redémarrage d’un nœud, le planificateur attend que le nombre défini de pods soit correctement planifié sur d’autres nœuds du cluster.

Examinons un exemple de jeu de réplicas avec cinq pods qui exécutent NGINX. Les pods dans le jeu de réplicas se voient affecter l’étiquette `app: nginx-frontend`. Durant un événement perturbateur volontaire comme une mise à niveau de cluster, vous devez veiller à ce qu’au moins trois pods s’exécutent. Le manifeste YAML suivant pour un objet *PodDisruptionBudget* définit ces exigences :

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
   name: nginx-pdb
spec:
   minAvailable: 3
   selector:
    matchLabels:
      app: nginx-frontend
```

Vous pouvez également définir un pourcentage (par exemple, *60 %* ), pour compenser automatiquement le scale-up du nombre de pods par le jeu de réplicas.

Vous pouvez définir le nombre maximal d’instances non disponibles dans un jeu de réplicas. Là encore, vous pouvez définir un pourcentage indiquant le nombre maximal de pods indisponibles. Selon le manifeste YAML du budget de perturbation de pod suivant, il ne peut pas y avoir plus de deux pods indisponibles dans le jeu de réplicas :

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
   name: nginx-pdb
spec:
   maxUnavailable: 2
   selector:
    matchLabels:
      app: nginx-frontend
```

Une fois votre budget de perturbation de pod défini, vous le créez dans votre cluster AKS comme tout autre objet Kubernetes :

```console
kubectl apply -f nginx-pdb.yaml
```

Collaborez avec les propriétaires et développeurs d’application pour comprendre leurs besoins et appliquer les budgets de perturbation de pod appropriés.

Pour plus d’informations sur l’utilisation de budgets de perturbation de pod, consultez [Spécifier un budget de perturbation pour votre application][k8s-pdbs].

## <a name="regularly-check-for-cluster-issues-with-kube-advisor"></a>Rechercher régulièrement les problèmes liés au cluster avec kube-advisor

**Meilleure pratique** : exécutez régulièrement la dernière version de l’outil open source `kube-advisor` pour détecter les problèmes dans votre cluster. Si vous appliquez des quotas de ressources sur un cluster AKS existant, exécutez d’abord `kube-advisor` pour rechercher les pods dont les demandes et limites de ressources ne sont pas définies.

L’outil [kube-advisor][kube-advisor] est un projet open source AKS associé qui analyse un cluster Kubernetes et signale les problèmes trouvés. Une vérification utile consiste à identifier les pods dépourvus de demandes et de limites de ressources.

L’outil kube-advisor peut rendre compte des limites et demandes de ressources manquantes dans PodSpecs pour les applications Windows et Linux, mais l’outil kube-advisor proprement dit doit être planifié sur un pod Linux. Vous pouvez planifier un pod pour qu’il s’exécute sur un pool de nœuds avec un système d’exploitation spécifique à l’aide d’un [sélecteur de nœud][k8s-node-selector] dans la configuration du pod.

Dans un cluster AKS qui héberge plusieurs applications et équipes de développement, il peut être difficile de suivre les pods sans ces demandes et limites de ressources. Une bonne pratique consiste à exécuter régulièrement `kube-advisor` sur vos clusters AKS, surtout si vous n’affectez pas de quotas de ressources aux espaces de noms.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons traité des fonctionnalités de base du planificateur Kubernetes. Pour plus d’informations sur les opérations liées aux clusters dans AKS, consultez les bonnes pratiques suivantes :

* [Multilocation et isolation de cluster][aks-best-practices-cluster-isolation]
* [Fonctionnalités avancées du planificateur Kubernetes][aks-best-practices-advanced-scheduler]
* [Authentification et autorisation][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-resource-quotas]: https://kubernetes.io/docs/concepts/policy/resource-quotas/
[configure-default-quotas]: https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
[kube-advisor]: https://github.com/Azure/kube-advisor
[k8s-pdbs]: https://kubernetes.io/docs/tasks/run-application/configure-pdb/

<!-- INTERNAL LINKS -->
[resource-limits]: developer-best-practices-resource-management.md#define-pod-resource-requests-and-limits
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
