---
title: Instances de conteneur et orchestration de conteneur
description: Découvrez comment se passe l’interaction entre Azure Container Instances et les orchestrateurs de conteneurs.
ms.topic: article
ms.date: 04/15/2019
ms.custom: mvc
ms.openlocfilehash: c7faeecfcc3a1d006cc923b48339a1242a0b9e9b
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148597"
---
# <a name="azure-container-instances-and-container-orchestrators"></a>Azure Container Instances et les orchestrateurs de conteneurs

En raison de leur taille réduite et de leur orientation d’application, les conteneurs conviennent parfaitement aux environnements de distribution agiles et aux architectures basées sur les microservices. La tâche d’automatisation et de gestion d’un grand nombre de conteneurs et de leur interaction porte le nom d’*orchestration*. Les orchestrateurs de conteneurs les plus courants sont Kubernetes, DC/OS et Docker Swarm.

Azure Container Instances fournit certaines des fonctionnalités de planification de base des plateformes d’orchestration. Et bien qu’il ne couvre pas les services de valeur plus élevée fournis par ces plateformes, Azure Container Instances peut leur être complémentaire. Cet article décrit l’étendue des services gérés par Azure Container Instances et explique comment les orchestrateurs de conteneurs complets peuvent interagir avec lui.

## <a name="traditional-orchestration"></a>Orchestration traditionnelle

La définition standard de l’orchestration comprend les tâches suivantes :

- **Planification** : avec une image conteneur et une requête de ressource données, rechercher une machine adaptée sur laquelle exécuter le conteneur.
- **Affinité/Anti-affinité** : spécifier que les conteneurs d’un ensemble doivent s’exécuter à proximité les uns des autres (à des fins de performances) ou suffisamment éloignés les uns des autres (à des fins de disponibilité).
- **Supervision de l’intégrité** : détecter les échecs de conteneur et les replanifier automatiquement.
- **Basculement** : effectuer le suivi de ce qui s’exécute sur chaque machine et replanifier des conteneurs à partir des machines ayant échoué sur des nœuds sains.
- **Mise à l’échelle** : ajouter ou supprimer des instances de conteneurs en fonction de la demande, manuellement ou automatiquement.
- **Réseau** : fournir un réseau superposé pour la coordination des conteneurs et la communication entre plusieurs machines hôtes.
- **Détection de service** : permettre aux conteneurs de se trouver mutuellement et automatiquement, même quand ils sont déplacés entre des machines hôtes et que leur adresse IP est modifiée.
- **Coordination des mises à niveau d’application** : gérer les mises à niveau des conteneurs pour éviter les temps d’arrêt des applications et autoriser une restauration en cas de problème.

## <a name="orchestration-with-azure-container-instances-a-layered-approach"></a>Orchestration avec Azure Container Instances : une approche en couches

Azure Container Instances permet d’adopter une approche en couches de l’orchestration. Vous bénéficiez ainsi de toutes les fonctionnalités de planification et de gestion nécessaires pour exécuter un seul conteneur, tout en permettant aux plateformes d’orchestration de gérer les tâches à conteneurs multiples.

Toute l’infrastructure sous-jacente pour Container Instances étant managée par Azure, une plateforme d’orchestration n’a pas besoin de rechercher une machine hôte appropriée sur laquelle exécuter un conteneur unique. L’élasticité du cloud garantit qu’un ordinateur est toujours disponible. Au lieu de cela, l’orchestrateur peut se concentrer sur les tâches qui simplifient le développement des architectures multi-conteneurs, notamment la mise à l’échelle et la coordination des mises à niveau.

## <a name="scenarios"></a>Scénarios

Bien que l’intégration d’orchestrateur avec Azure Container Instances en soit encore à sa phase initiale, nous prévoyons l’émergence de plusieurs environnements différents :

### <a name="orchestration-of-container-instances-exclusively"></a>Orchestration exclusive de Container Instances

Du fait du démarrage rapide et de la facturation à la seconde, un environnement basé exclusivement sur Azure Container Instances offre le moyen le plus rapide de bien démarrer et de faire face à des charges de travail très variables.

### <a name="combination-of-container-instances-and-containers-in-virtual-machines"></a>Combinaison de Container Instances et de conteneurs sur des machines virtuelles

Pour les charges de travail longues et stables, l’orchestration des conteneurs dans un cluster de machines virtuelles dédiées est généralement moins coûteuse que l’exécution des mêmes conteneurs avec Azure Container Instances. Toutefois, Container Instances offre une excellente solution pour développer et contracter rapidement votre capacité globale en cas de pics d’utilisation inattendus ou de courte durée.

Au lieu de mettre à l’échelle le nombre de machines virtuelles dans votre cluster et de déployer ensuite des conteneurs supplémentaires sur ces machines, l’orchestrateur peut simplement planifier les conteneurs supplémentaires à l’aide d’Azure Container Instances et les supprimer quand ils ne sont plus nécessaires.

## <a name="sample-implementation-virtual-nodes-for-azure-kubernetes-service-aks"></a>Exemple d’implémentation : nœuds virtuels pour Azure Kubernetes Service (AKS)

Pour rapidement mettre à l’échelle des charges de travail dans un cluster [Azure Kubernetes Service](../aks/intro-kubernetes.md) (AKS), vous pouvez utiliser des *nœuds virtuels* créés dynamiquement dans Azure Container Instances. Les nœuds virtuels permettent la communication réseau entre les pods qui s’exécutent dans ACI et le cluster AKS. 

Les nœuds virtuels prennent actuellement en charge les instances de conteneur Linux. Découvrez les nœuds virtuels avec [Azure CLI](../aks/virtual-nodes-cli.md) ou le [Portail Azure](../aks/virtual-nodes-portal.md).

Les nœuds virtuels utilisent [Virtual Kubelet][aci-connector-k8s] open source pour imiter le [kubelet][kubelet-doc] dans Kubernetes en s’inscrivant en tant que nœud avec une capacité illimitée. Le kubelet virtuel distribue la création de [pods][pod-doc] en tant que groupes de conteneurs dans Azure Container Instances.

Consultez le projet [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet) pour obtenir des exemples supplémentaires de l’extension de l’API Kubernetes aux plateformes de conteneurs serverless.

## <a name="next-steps"></a>Étapes suivantes

Créez votre premier conteneur avec Azure Container Instances à l’aide du [guide de démarrage rapide](container-instances-quickstart.md).

<!-- IMAGES -->

<!-- LINKS -->
[aci-connector-k8s]: https://github.com/virtual-kubelet/azure-aci
[kubelet-doc]: https://kubernetes.io/docs/admin/kubelet/
[pod-doc]: https://kubernetes.io/docs/concepts/workloads/pods/pod/