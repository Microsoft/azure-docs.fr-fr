---
title: Consulter les journaux d’activité kubelet dans Azure Kubernetes Service (AKS)
description: Découvrez comment afficher des informations de dépannage dans les journaux kubelet à partir des nœuds du service AKS (Azure Kubernetes Service)
services: container-service
ms.topic: article
ms.date: 03/05/2019
ms.openlocfilehash: 2ef38aa76f4ef9152e7bc55a1d74c84ef426f0ac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87056761"
---
# <a name="get-kubelet-logs-from-azure-kubernetes-service-aks-cluster-nodes"></a>Accéder aux journaux d’activité kubelet à partir de nœuds serveur Azure Kubernetes Service (AKS)

Dans le cadre de l’utilisation d’un cluster AKS, vous devrez peut-être consulter les journaux pour résoudre un problème. La possibilité de voir les journaux pour les [composants maîtres AKS][aks-master-logs] ou [conteneurs dans un cluster AKS][azure-container-logs] est intégrée au portail Azure. Il vous arrive parfois de devoir récupérer les journaux *kubelet* à partir d’un nœud AKS à des fins de dépannage.

Cet article vous montre comment vous pouvez utiliser `journalctl` pour afficher les journaux *kubelet* sur un nœud AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

## <a name="create-an-ssh-connection"></a>Créer une connexion SSH

Commencez par créer une connexion SSH avec le nœud duquel vous avez besoin de voir les journaux d’activité de *kubelet*. Cette opération est détaillée dans le document [SSH dans les nœuds de cluster Azure Kubernetes Service (AKS)][aks-ssh].

## <a name="get-kubelet-logs"></a>Obtenir des journaux d’activité kubelet

Une fois connecté au nœud, exécutez la commande suivante pour récupérer les journaux d’activité *kubelet* :

```console
sudo journalctl -u kubelet -o cat
```

> [!NOTE]
> Pour les nœuds Windows, les données du journal se trouvent dans `C:\k` et peuvent être consultées à l’aide de la commande *more* :
> ```
> more C:\k\kubelet.log
> ```

L’exemple de sortie suivant indique que les données de journal *kubelet* :

```
I0508 12:26:17.905042    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:27.943494    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:28.920125    8672 server.go:796] GET /stats/summary: (10.370874ms) 200 [[Ruby] 10.244.0.2:52292]
I0508 12:26:37.964650    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:47.996449    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:58.019746    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:05.107680    8672 server.go:796] GET /stats/summary/: (24.853838ms) 200 [[Go-http-client/1.1] 10.244.0.3:44660]
I0508 12:27:08.041736    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:18.068505    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:28.094889    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:38.121346    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:44.015205    8672 server.go:796] GET /stats/summary: (30.236824ms) 200 [[Ruby] 10.244.0.2:52588]
I0508 12:27:48.145640    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:58.178534    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:05.040375    8672 server.go:796] GET /stats/summary/: (27.78503ms) 200 [[Go-http-client/1.1] 10.244.0.3:44660]
I0508 12:28:08.214158    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:18.242160    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:28.274408    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:38.296074    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:48.321952    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:58.344656    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
```

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’informations de dépannage supplémentaires à partir du maître Kubernetes, consultez [Afficher les journaux d’activité de nœud principal Kubernetes dans AKS][aks-master-logs].

<!-- LINKS - internal -->
[aks-ssh]: ssh.md
[aks-master-logs]: view-master-logs.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-master-logs]: view-master-logs.md
[azure-container-logs]: ../azure-monitor/insights/container-insights-overview.md
