---
title: Haute disponibilité pour les conteneurs Azure SQL Edge - Azure SQL Edge
description: Découvrez ce qu’est la haute disponibilité pour les conteneurs Azure SQL Edge
keywords: SQL Edge, conteneurs, haute disponibilité
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 56b9f06547f737bc05d573f98ce1dbac2ba48758
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929907"
---
# <a name="high-availability-for-azure-sql-edge-containers"></a>Haute disponibilité pour les conteneurs Azure SQL Edge

Créez et gérez vos instances Azure SQL Edge en mode natif dans Kubernetes. Déployez Azure SQL Edge sur des conteneurs Docker managés par [Kubernetes](https://kubernetes.io/). Dans Kubernetes, un conteneur avec une instance Azure SQL Edge peut être récupéré automatiquement en cas de défaillance d’un nœud de cluster. Vous pouvez configurer l’image conteneur SQL Edge avec une revendication de volume persistant Kubernetes (PVC). Kubernetes supervise le processus Azure SQL Edge dans le conteneur. En cas d’échec du processus, du pod, du conteneur ou du nœud, Kubernetes démarre automatiquement une autre instance et se reconnecte au stockage.

## <a name="azure-sql-edge-containers-on-kubernetes"></a>Conteneurs Azure SQL Edge sur Kubernetes

Kubernetes 1.6 et ultérieur prend en charge les [*classes de stockage*](https://kubernetes.io/docs/concepts/storage/storage-classes/) et les [*revendications de volume persistant*](https://kubernetes.io/docs/concepts/storage/storage-classes/#persistentvolumeclaims).

Dans cette configuration, Kubernetes joue le rôle de l’orchestrateur de conteneur. 

![Diagramme d’Azure SQL Edge dans un cluster Kubernetes](media/deploy-kubernetes/kubernetes-sql-edge.png)

Dans le diagramme précédent, `azure-sql-edge` est un conteneur dans un [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). Kubernetes orchestre les ressources dans le cluster. Un [jeu de réplicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) garantit que le pod est automatiquement récupéré après une défaillance de nœud. Les applications se connectent au service. Dans ce cas, le service représente un équilibreur de charge qui héberge une adresse IP restant la même après la défaillance de `azure-sql-edge`.

Dans le diagramme suivant, le conteneur `azure-sql-edge` a échoué. En tant qu’orchestrateur, Kubernetes garantit le nombre correct d’instances saines dans le jeu de réplicas et démarre un nouveau conteneur en fonction de la configuration. L’orchestrateur démarre un nouveau pod sur le même nœud et `azure-sql-edge` se reconnecte au même stockage persistant. Le service se connecte à `azure-sql-edge` recréé.

![Azure SQL Edge dans un cluster Kubernetes après l’échec du pod](media/deploy-kubernetes/kubernetes-sql-edge-after-pod-fail.png)

Dans le diagramme suivant, le nœud hébergeant le conteneur `azure-sql-edge` a échoué. L’orchestrateur démarre un nouveau pod sur un nœud différent et `azure-sql-edge` se reconnecte au même stockage persistant. Le service se connecte à `azure-sql-edge` recréé.

![Azure SQL Edge dans un cluster Kubernetes après la défaillance du nœud](media/deploy-kubernetes/kubernetes-sql-edge-after-node-fail.png).

Pour créer un conteneur dans Kubernetes, consultez [Déployer un conteneur Azure SQL Edge dans Kubernetes](deploy-Kubernetes.md)

## <a name="next-steps"></a>Étapes suivantes

Pour déployer des conteneurs Azure SQL Edge dans AKS (Azure Kubernetes Service), consultez les articles suivants :
- [Déployer un conteneur Azure SQL Edge dans Kubernetes](deploy-Kubernetes.md)
- [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md).
- [Création d’une solution IoT de bout en bout avec SQL Edge à l’aide d’IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de données dans Azure SQL Edge](stream-data.md)