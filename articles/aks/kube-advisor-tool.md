---
title: Rechercher les bonnes pratiques dans les déploiements
titleSuffix: Azure Kubernetes Service
description: Découvrez comment vérifier l’implémentation des meilleures pratiques dans vos déploiements sur Azure Kubernetes Service à l’aide de kube-advisor
services: container-service
author: seanmck
ms.topic: troubleshooting
ms.date: 11/05/2018
ms.author: seanmck
ms.openlocfilehash: 7730146f30487eb5d20f0d3138e9e5ba799daa99
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681514"
---
# <a name="checking-for-kubernetes-best-practices-in-your-cluster"></a>Vérification des meilleures pratiques Kubernetes dans votre cluster

Il existe plusieurs pratiques recommandées que vous devez suivre pour vos déploiements Kubernetes afin de garantir des performances et une résilience optimales pour vos applications. Vous pouvez utiliser l’outil kube-advisor pour rechercher les déploiements qui ne suivent pas ces suggestions.

## <a name="about-kube-advisor"></a>À propos de kube-advisor

L’[outil kube-advisor][kube-advisor-github] est un conteneur unique conçu pour être exécuté sur votre cluster. Il interroge le serveur d’API Kubernetes pour obtenir des informations sur vos déploiements et retourne un ensemble de suggestions d’améliorations.

L’outil kube-advisor peut rendre compte des limites et demandes de ressources manquantes dans PodSpecs pour les applications Windows et Linux, mais l’outil kube-advisor proprement dit doit être planifié sur un pod Linux. Vous pouvez planifier un pod pour qu’il s’exécute sur un pool de nœuds avec un système d’exploitation spécifique à l’aide d’un [sélecteur de nœud][k8s-node-selector] dans la configuration du pod.

> [!NOTE]
> L’outil kube-advisor est pris en charge par Microsoft dans la mesure du possible. Les problèmes et suggestions doivent être déposés sur GitHub.

## <a name="running-kube-advisor"></a>Exécution de kube-advisor

Pour exécuter l’outil sur un cluster configuré pour le [contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes)](./azure-ad-integration-cli.md), à l’aide des commandes suivantes. La première commande crée un compte de service Kubernetes. La deuxième commande exécute l’outil dans un pod à l’aide de ce compte de service et configure le pod pour qu’il soit supprimé après sa fermeture. 

```bash
kubectl apply -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml

kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never --overrides="{ \"apiVersion\": \"v1\", \"spec\": { \"serviceAccountName\": \"kube-advisor\" } }" --namespace default
```

Si vous n’utilisez pas le contrôle d’accès en fonction du rôle (RBAC Kubernetes), vous pouvez exécuter la commande comme suit :

```bash
kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never
```

Après quelques secondes, vous devez voir une table décrivant les améliorations potentielles de vos déploiements.

![Sortie de kube-advisor](media/kube-advisor-tool/kube-advisor-output.png)

## <a name="checks-performed"></a>Vérifications effectuées

L’outil valide les meilleures pratiques Kubernetes suivantes, chacune ayant ses propres suggestions de correction.

### <a name="resource-requests-and-limits"></a>Demandes et limites de ressources

Kubernetes prend en charge la définition des [demandes et limites de ressources pour les spécifications de pod][kube-cpumem]. La requête définit le nombre de processeurs et la mémoire minimums nécessaires pour exécuter le conteneur. La limite définit le nombre de processeurs et la mémoire maximums devant être autorisés.

Par défaut, aucune demande ni aucune limite ne sont définies dans les spécifications de pod. Cela peut entraîner une surplanification des nœuds et une sous-alimentation des conteneurs. L’outil kube-advisor met en évidence les pods sans demande ou limite définies.

## <a name="cleaning-up"></a>Nettoyage

Si le contrôle d’accès en fonction du rôle (RBAC Kubernetes) est activé sur votre cluster, vous pouvez nettoyer le `ClusterRoleBinding` après avoir exécuté l’outil à l’aide de la commande suivante :

```bash
kubectl delete -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml
```

Si vous exécutez l’outil sur un cluster dont RBAC Kubernetes n’est pas activé, aucun nettoyage n’est requis.

## <a name="next-steps"></a>Étapes suivantes

- [Résoudre les problèmes liés à Azure Kubernetes Service](troubleshooting.md)

<!-- RESOURCES -->

[kube-cpumem]: https://github.com/Azure/azure-quickstart-templates
[kube-advisor-github]: https://github.com/azure/kube-advisor
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
